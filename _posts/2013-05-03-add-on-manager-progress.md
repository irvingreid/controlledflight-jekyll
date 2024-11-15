---
title: 'Add-On Manager Progress'
date: '2013-05-03T11:53:24-04:00'
author: irving
layout: post
permalink: /2013/05/03/add-on-manager-progress/
categories:
    - Mozilla
---

It’s been a month since [Felipe](https://twitter.com/felipc) and I started coding on the add-on manager conversion, and we’ve made quite a bit of progress. The core code for both modules has been converted to load and save in JSON format; we’re now cleaning up corner cases like upgrading from previous database formats.

By far the biggest headache for me has been debugging test failures in the [xpcshell](https://developer.mozilla.org/en/docs/Writing_xpcshell-based_unit_tests "Writing xpcshell-based unit tests") test suite. The test suite for Addon Manager is extremely thorough, which is great because it gives me quite a bit of confidence that the new version will work correctly if we get all the tests to pass. The down side is that almost all of the tests are written as end-to-end scenarios where a number of add-ons are installed, updated, uninstalled, etc. and the (simulated) browser environment is started, stopped, upgraded, etc. When a test fails, there is usually an extended debugging session necessary to find the cause.

Unfortunately, all our spiffy new JavaScript debugging tools don’t work in the xpcshell environment; we really need to make progress on [Bug 809561](https://bugzilla.mozilla.org/show_bug.cgi?id=809561 "NEW - Integrate xpcshell test harness with chrome remote debugging") so that we aren’t stuck with putting dump() statements all over the code to find problems. I did put together one patch for the asynchronous test harness to print function names as asynchronous test cases start and end, to at least make it easier to diagnose hanging async tests; that’s in [bug 863311](https://bugzilla.mozilla.org/show_bug.cgi?id=863311 "ASSIGNED - Add test function names to "Test pending" / "Test complete" messages").

See [Bug 853388](https://bugzilla.mozilla.org/show_bug.cgi?id=853388 "Bug 853388 - Convert XPIProvider.jsm from sqlite to JSON") and [Bug 853389](https://bugzilla.mozilla.org/show_bug.cgi?id=853389 "Bug 853389 - Convert AddonRepository.jsm from sqlite to JSON") for work-in-progress patches and discussion of some of the other issues that have come up.
