---
id: 107
title: 'Saving browser state asynchronously'
date: '2013-05-31T15:24:24-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/?p=107'
permalink: /2013/05/31/saving-browser-state-asynchronously/
categories:
    - Mozilla
    - Software
---

As part of switching the Firefox extensions databases from sqlite to a flat file containing JSON, we want to build a module that flushes the in-memory state of the data out to disk after it changes, in a way that doesn’t hang the main thread waiting for the I/O to complete. Felipe did an initial implementation for [bug 853389](https://bugzilla.mozilla.org/show_bug.cgi?id=853389 "Bug 853389 - Convert AddonRepository.jsm from sqlite to JSON") based on the [DeferredTask](https://mxr.mozilla.org/mozilla-central/source/toolkit/modules/DeferredTask.jsm) module, and I’ve been working on sorting out all the edge cases that the XPIDatabase tests trigger. Unfortunately, there are plenty of edge cases. As a result the attempted implementation is getting rather twisted, so I wanted to put up the current state of my design both to clarify it in my own mind, and to solicit input on ways we could simplify. In particular, I’d like to hear from people who have a good feel for [Promise](https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Promise.jsm "Promise.jsm - Mozilla | MDN") based asynchronous code, to see if there’s an idiomatic Promise based implementation that is more straightforward than the current approach with explicit callbacks.

The basic assumptions of the design are:

- The working copy of the data is a JavaScript tree is mapped to and from the on-disk structure.
- All modification of the data is done through the JS structure, so the data only needs to be read in once at start up time.
- When we decide to write the data, we take a snapshot (via JSON.stringify) and then start the async I/O.
- There is no locking; the in-memory data can be modified while we’re saving the snapshot.
- We wait for a short time after the in-memory data is modified before we start writing, so that we can coalesce sets of changes into a single write.
- Clients indicate that the data needs to be saved by calling saveChanges(), and can optionally be notified (by callback or Promise resolution) when all in-memory changes up to the time saveChanges() was called have been written to disk.

The core states for data saving are:

1. In memory data is clean and synced to disk
2. In memory data is dirty, waiting before syncing snapshot to disk
3. Memory clean, writing of snapshot in progress
4. Memory dirty, writing of snapshot in progress

With the API that Felipe built for AddonRepository, we support clients providing a callback when they mark the data dirty, and that callback will be invoked after the change in question is written to disk.

In normal operation, the state transitions are:

Clean/Synced –&gt; (modify data) –&gt; Dirty/Waiting  
Dirty/Waiting –&gt; (write starts) –&gt; Clean/InProgress  
Clean/InProgress –&gt; (write ends) –&gt; Clean/Synced

When operations overlap, we see:

Dirty/Waiting –&gt; (modify data) –&gt; Dirty/Waiting  
Clean/InProgress –&gt; (modify data) –&gt; Dirty/InProgress  
Dirty/InProgress –&gt; (modify data) –&gt; Dirty/InProgress

Dirty/InProgress –&gt; (write ends) –&gt; Dirty/Waiting

The constraints this meets are:

- We only have one write going at a time
- Data could be modified at any time
- Multiple modifications can be batched into a single write (though the delay could be as short as the next event loop cycle)
- Beginning to write the data takes a snapshot (JSON.stringify) so it’s OK for the in-memory structure to change while the write is in progress
- If the data is modified while we’re writing, we need to write again after the in progress write completes

The presence of the callback makes the control flow a little more complicated; for example, we support the following sequence of events:

saveChanges(callback\_1)Â (now Dirty/Waiting)  
saveChanges(callback\_1b) (still Dirty/Waiting)  
(possible delay before batched write)  
begin writingÂ Â Â Â Â Â Â Â Â Â Â (now Clean/InProgress)  
saveChanges(callback\_2)Â (now Dirty/InProgress)  
SaveChanges(callback\_2b) (still Dirty/InProgress)  
end writing  
callback\_1(result)  
callback\_1b(result)Â Â Â Â Â (now Dirty/Waiting)  
(possible delay before batched write)  
begin writingÂ Â Â Â Â Â Â Â Â Â Â (now Clean/InProgress)  
end writing  
callback\_2(result)  
callback\_2b(result)Â Â Â Â Â (now Clean/Synced)

Turns out we may be able to drop the callback from saveChanges, because neither AddonRepository nor XPIDatabase need it. It saves a bit of bookkeeping, but I’m not sure it makes a huge difference to the overall complexity.

The big headache comes in flushing the data at shutdown time. We need to skip all the delays and get the data flushed, and because things are happening asynchronously, we need to be careful that objects we need aren’t destroyed before we use them. We also need to support a final callback to signal the completion of the flush. We assume that it is unsupported to modify the in-memory data once a flush starts; whether we throw an error if someone tries is TBD.

The flows we need here are:

Clean/Synced:

immediately call flush\_callback(success)

Dirty/Waiting:

Cancel batched write delay  
begin writing  
(asynchronous wait)  
end writing  
call saveChanges callbacks  
call flush\_callback

Clean/InProgress:

(async wait)  
end writing  
call saveChanges callbacks  
call flush\_callback

Dirty/InProgress: It’s easiest to start with the scenario described above for normal operation…

saveChanges(callback\_1)Â (now Dirty/Waiting)  
saveChanges(callback\_1b) (still Dirty/Waiting)  
(possible delay before batched write)  
begin writingÂ Â Â Â Â Â Â Â Â Â Â (now Clean/InProgress)  
saveChanges(callback\_2)Â (now Dirty/InProgress)  
SaveChanges(callback\_2b) (still Dirty/InProgress)  
\*\*\* flush(flush\_callback)  
capture snapshot of in-memory data  
(async wait)  
end writing  
callback\_1(result)  
callback\_1b(result)  
\*\*\* no delay  
begin writing saved snapshot  
(async wait)  
end writing  
callback\_2(result)  
callback\_2b(result)  
flush\_callback(result)

The important differences in the shutdown time flow are:

- Capture the in-memory state immediately, rather than waiting for the “begin writing” event – this protects against the state being cleaned up by other shutdown events while we’re waiting for the first async write to complete.
- No delay between the end of the in-progress write and the start of the second write.
- We could do the final write synchronously, since it has to be complete before Firefox exits, but the flush API would still need to be async to allow for an async write that was already in progress.

At some point we’d like to wrap all this in a module that other parts of the browser could use. First we want to get the behaviour right…