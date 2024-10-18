---
id: 146
title: 'A Mixed Bag of Telemetry'
date: '2014-03-20T16:59:59-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/?p=146'
permalink: /2014/03/20/a-mixed-bag-of-telemetry/
categories:
    - Mozilla
---

There were a few open questions, and follow ups on previous work, that I wanted to look at in Mozilla’s Telemetry database (<https://wiki.mozilla.org/Telemetry>, though that page is out of date).

## Add-on Compatibility Check

For [**BugÂ 772484**](https://bugzilla.mozilla.org/show_bug.cgi?id=772484 "Bug 772484 - extension check dialog is annoying and can effectively hang the Firefox process"), we wanted to know how often browser start up is blocked by the add-on compatibility check. Fortunately there is a clear telemetry probe for that; we set SIMPLE\_MEASURES\_STARTUPINTERRUPED to 1 if we display the dialog. Looking at the current (as of last week) Release and Beta versions in the dashboard (e.g. [http://telemetry.mozilla.org/#release/27/SIMPLE\_MEASURES\_STARTUPINTERRUPTED](http://telemetry.mozilla.org/#release/27/SIMPLE_MEASURES_STARTUPINTERRUPTED))

Startup Interrupted (millions of events)
| Version | Sessions | No | Yes | % |
|---|---|---|---|---|
| Release 27 | 318.96 | 313.07 | 5.89 | 1.85 |
| Beta 28 | 11.66 | 11.53 | 0.128 | 1.10 |

Now, we only show this UI if (a) the version string of the installed browser changed from the last run in any way (major or minor version change) and (b) the user has add-ons installed somewhere other than the browser’s installation directory; that is, add-ons that weren’t included as part of the software distribution and thus would be outside the user’s control. So that’s the obvious next question – how often does a browser version change \*not\* trigger the compatibility dialog? This needed a custom Telemetry analysis (<https://github.com/irvingreid/addon-telemetry/blob/master/startup-perf/interrupted-mr.py>). I sampled a few days around when Fx 27 was released, when most users would be upgrading.

Version Changes
| Dates | Sessions | Version changed | Interrupted | % |
|---|---|---|---|---|
| Feb 4-8 | 6653299 | 959662 | 662146 | 69.0 |
| Feb 10-11 | 8333667 | 291936 | 215213 | 73.7 |

We’ll want to run this (or a comparable analysis) after [**BugÂ 760356**](https://bugzilla.mozilla.org/show_bug.cgi?id=760356 "Bug 760356 - Only show the add-on compatibility UI when actually necessary") to see if we’ve improved.

## SQLITE to JSON conversions

Did [**BugÂ 853388**](https://bugzilla.mozilla.org/show_bug.cgi?id=853388) and [**BugÂ 853389**](https://bugzilla.mozilla.org/show_bug.cgi?id=853389) make a noticeable difference in browser start up time? For this, I looked at the processed data in the new Telemetry dashboard at <http://telemetry.mozilla.org/> for both Firefox (the desktop browser) and Fennec (the Android browser). Bug 853389 shipped in Firefox 25, and 853388 shipped in Fx 26. For the desktop browser we normally measure start up time using ‘FIRSTPAINT’, which is the number of milliseconds from when firefox.exe started running, to when we start displaying the first web page. The following table is summarized from [http://telemetry.mozilla.org/#release/24/SIMPLE\_MEASURES\_FIRSTPAINT](http://telemetry.mozilla.org/#release/24/SIMPLE_MEASURES_FIRSTPAINT) (and ..25.. etc. for the other releases):

Firefox Release, Time to First Paint (seconds)
| Percentile | Version |
|---|---|
|  | 24 | 25 | 26 | 27 |
| 5% | 0.692 | 0.695 | 0.694 | 0.693 |
| 25% | 1.52 | 1.53 | 1.52 | 1.52 |
| median | 3.12 | 3.37 | 2.98 | 3.22 |
| 75% | 7.13 | 7.46 | 6.51 | 7.41 |
| 95% | NaN | NaN | NaN | 30.01 |

Firefox Beta, Time to First Paint (seconds)
| Percentile | Version |
|---|---|
|  | 24 | 25 | 26 | 27 |
| 5% | 0.714 | 0.714 | 0.711 | 0.710 |
| 25% | 1.81 | 1.79 | 1.59 | 1.59 |
| median | 3.58 | 3.47 | 3.39 | 3.40 |
| 75% | 8.00 | 7.63 | 7.49 | 7.49 |
| 95% | NaN | 30.03 | 30.01 | 30.01 |

For the Android browser (Fennec), we don’t collect FIRSTPAINT but we have an event FENNEC\_STARTUP\_TIME\_GECKOREADY that records when the HTML &amp; JavaScript engine is done initializing.

Fennec Release, Time to Gecko Ready (seconds)
| Percentile | Version |
|---|---|
|  | 24 | 25 | 26 | 27 |
| 5% | 1.72 | 1.72 | 2.10 | 2.10 |
| 25% | 2.41 | 2.58 | 2.75 | 2.68 |
| median | 3.18 | 3.19 | 3.90 | 3.90 |
| 75% | 4.83 | 4.85 | 6.02 | 6.09 |
| 95% | 13.32 | 13.34 | 16.39 | 16.41 |

Unfortunately there isn’t a really clear signal in this data; the higher percentiles of Desktop do improve a little; this makes sense, since browser profiles with extensions installed are likely to have a longer start up time to begin with and be more affected by the change in data storage. The Fennec start up times get significantly worse in version 26; we’re not sure why yet, it could be the overhead of IO.File starting up a separate JavaScript worker thread.

## Start-up Exceptions

[Bug 952543 ](https://bugzilla.mozilla.org/show_bug.cgi?id=952543 "Report startup exceptions in AddonManager and XPIProvider through telemetry") added telemetry reporting of exceptions in Addon Manager and XPI Provider start up, and [BugÂ 972852](https://bugzilla.mozilla.org/show_bug.cgi?id=972852 "Startup exceptions in AddonManager / XPIProvider") fixed several of the bugs revealed. I re-ran the analysis; the fixes in 972853 worked, but there are still a few issues. Filed &amp; patched [BugÂ 986080](https://bugzilla.mozilla.org/show_bug.cgi?id=986080) and [BugÂ 986000](https://bugzilla.mozilla.org/show_bug.cgi?id=986000); filed [BugÂ 985998](https://bugzilla.mozilla.org/show_bug.cgi?id=985998 "Exceptions updating preferences in AddonManager and XPIProvider")<span id="summary_alias_container"><span id="short_desc_nonedit_display"> and [started a discussion on mozilla.dev.platform about the Preferences API](https://groups.google.com/forum/#!topic/mozilla.dev.platform/lB38FhUGH-s "Should nsIPrefBranch.set*Pref return NS_ERROR_UNEXPECTED on type mismatch?"), and filed </span></span>[BugÂ 986104](https://bugzilla.mozilla.org/show_bug.cgi?id=986104 "Unexplained failures to start XPIProvider").<span id="summary_alias_container"><span id="short_desc_nonedit_display"></span>  
</span>