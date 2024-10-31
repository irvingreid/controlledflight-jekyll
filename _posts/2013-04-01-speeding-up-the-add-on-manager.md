---
id: 74
title: 'Speeding up the Add-On Manager'
date: '2013-04-01T11:52:33-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/?p=74'
permalink: /2013/04/01/speeding-up-the-add-on-manager/
categories:
    - Mozilla
    - Software
---

One of the focuses of the performance team this year is to improve start-up time for the browser. We’ve identified that the Add-on Manager has several issues that can delay start up, and can also cause brief user interface hangs (“jank”) during normal operation. [This Bugzilla search](https://bugzilla.mozilla.org/buglist.cgi?list_id=6163136&resolution=---&resolution=DUPLICATE&status_whiteboard_type=allwordssubstr&query_format=advanced&status_whiteboard=snappy&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=READY&bug_status=ASSIGNED&bug_status=REOPENED&component=Add-ons%20Manager) shows what we’re tracking right now.

After investigating [bug 729330](https://bugzilla.mozilla.org/show_bug.cgi?id=729330 "Bug 729330 - Addon-manager jank: RELEASE SAVEPOINT 'default' is #1 main thread SQL query >100ms on nightly"), we decided that the best approach to removing the database I/O bottlenecks in the code was to remove the database. The amount of information we store about addons is relatively small (typically tens of kilobytes, a few hundred KB in the worst case). It is not modified very often, and doesn’t really require the sort of random search and update provided by the SQLITE database we’re currently using. We hope that switching the data storage to JSON, with asynchronous I/O to update the persistent copy in the user’s profile, will improve both overall performance and responsiveness.

We hashed out the approach on an Etherpad document at <https://etherpad.mozilla.org/snappy-addon-manager>, and now [Felipe](https://twitter.com/felipc) and I are digging in and starting to change code. The work is being done under [BugÂ 853388](https://bugzilla.mozilla.org/show_bug.cgi?id=853388 "Bug 853388 - Convert XPIProvider.jsm from sqlite to JSON") and [BugÂ 853389](https://bugzilla.mozilla.org/show_bug.cgi?id=853389 "Bug 853389 - Convert AddonRepository.jsm from sqlite to JSON").

I’ll post progress reports here every week or two, or when anything particularly interesting comes up.