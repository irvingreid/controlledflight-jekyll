---
title: 'How many Firefox extensions do people use?'
date: '2014-06-02T22:33:55-04:00'
author: irving
layout: post
permalink: /2014/06/02/how-many-firefox-extensions-do-people-use/
categories:
    - Mozilla
---

I found myself wondering what a “normal” number of XPI extensions is, and since I was working on some other analysis runs on the Mozilla Telemetry database, I took the time to run a count across one day’s data (May 29 2014). Without much further ado, here’s the median and higher percentiles of the number of active add-ons for Firefox and Fennec (Firefox for Android):

Installed add-ons, all versions/channels
| Application | Platform | Sessions | Median | 75 % | 90 % | 95 % | 99 % | max |
|---|---|---|---|---|---|---|---|---|
| Firefox | WINNT | 10472433 | 3 | 5 | 8 | 11 | 19 | 273 |
| Firefox | Darwin | 364473 | 3 | 6 | 9 | 12 | 21 | 117 |
| Firefox | Linux | 22701 | 4 | 8 | 16 | 23 | 35 | 112 |
| Firefox | ALL | 10859607 | 3 | 5 | 8 | 11 | 19 | 273 |
| Fennec | Android | 735437 | 0 | 0 | 2 | 3 | 6 | 36 |

Of course there are lots of caveats… These are only enabled add-ons; there was a bug in my code to collect total installed add-ons, and I didn’t feel like waiting for another multi-hour run to get the total. These are only the add-ons managed by the XPI Provider, which means Extensions, Dictionaries and Themes (but not Lightweight Themes or Plugins). Desktop versions of Firefox all come with a pre-configured Default theme, but Fennec doesn’t. Telemetry counts sessions, not users, so the results are somewhat biased toward users that restart their browser more often, and for many users Telemetry is opt-in, so the results are also biased toward the sort of users who would enable it.

That said, we have a useful estimate of how large our internal data structures will get, which can help us make implementation decisions.

The Telemetry map-reduce I used for this had a couple of bugs that I worked around to get my output; because it was a one-off I didn’t bother fixing them and re-running the analysis. That said, you can see the code, map-reduce output and further summarized data at <https://github.com/irvingreid/addon-telemetry/tree/master/addon-count>.
