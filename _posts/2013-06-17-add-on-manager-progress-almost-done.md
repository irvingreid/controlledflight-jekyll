---
id: 117
title: 'Add-on Manager progress: Almost done!'
date: '2013-06-17T10:05:42-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/?p=117'
permalink: /2013/06/17/add-on-manager-progress-almost-done/
categories:
    - Mozilla
    - Software
---

Felipe has a full suite of r+ for his work on AddonRepository.jsm in [BugÂ 853389](https://bugzilla.mozilla.org/show_bug.cgi?id=853389 "Bug 853389 - Convert AddonRepository.jsm from sqlite to JSON"), and I’m in the middle of handling review comments for the XPI Database changes in [BugÂ 853388](https://bugzilla.mozilla.org/show_bug.cgi?id=853388 "Bug 853388 - Convert XPIProvider.jsm from sqlite to JSON"). I need to update based on the review comments, implement asynchronous loading of the JSON database, and add some telemetry so we can track performance and correctness of the new version. Once that is done, we’ll be ready to land on Nightly.

I’ve implemented a DeferredSave module based in the discussion in my previous blog post; I suspect it’s something that will be useful to others trying to convert to asynchronous saving of data blobs. Check it out at https://bugzilla.mozilla.org/attachment.cgi?id=762461.