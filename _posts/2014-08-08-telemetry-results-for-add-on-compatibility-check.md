---
id: 165
title: 'Telemetry Results for Add-on Compatibility Check'
date: '2014-08-08T11:30:18-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/?p=165'
permalink: /2014/08/08/telemetry-results-for-add-on-compatibility-check/
categories:
    - Mozilla
    - Software
---

Earlier this year (in Firefox 32), we landed a fix for [bug 760356](https://bugzilla.mozilla.org/show_bug.cgi?id=760356 "Bug 760356 - Only show the add-on compatibility UI when actually necessary"), to reduce how often we delay starting up the browser in order to check whether all your add-ons are compatible. We landed the related [bug 1010449](https://bugzilla.mozilla.org/show_bug.cgi?id=1010449 "Bug 1010449 - Implement telemetry measures from bug 760356 for before/after analysis") in Firefox 31 to gather telemetry about the compatibility check, so that we could to before/after analysis.

## Background

When you upgrade to a new version of Firefox, changes to the core browser can break add-ons. For this reason, every add-on comes with metadata that says which versions of Firefox it works with. There are a couple of straightforward cases, and quite a few tricky corners…

- The add-on is compatible with the new Firefox, and everything works just fine.
- The add-on is incompatible and must be disabled. 
    - But maybe there’s an updated version of the add-on available, so we should upgrade it.
    - Or maybe the add-on was installed in a system directory by a third party (e.g. an antivirus toolbar) and Firefox can’t upgrade it.
- The add-on says it’s compatible, but it’s not – this could break your browser! 
    - The add-on author could discover this in advance and publish updated metadata to mark the add-on incompatible.
    - Mozilla could discover the incompatibility and publish a metadata override at addons.mozilla.org to protect our users.
- The add-on says it’s not compatible, but it actually is. 
    - Again, either the add-on author or Mozilla can publish a compatibility override.

We want to keep as many add-ons as possible enabled, because our users love (most of) their add-ons, while protecting users from incompatible add-ons that break Firefox. To do this, we implemented a very conservative check every time you update to a new version. On the first run with a new Firefox version, before we load any add-ons we ask addons.mozilla.org \*and\* each add-on’s update server whether there is a metadata update available, and whether there is a newer version of the add-on compatible with the new Firefox version. We then enable/disable based on that updated metadata, and offer the user the chance to upgrade those add-ons that have new versions available. Once this is done, we can load up the add-ons and finish starting up the browser.

This check involves multiple network requests, so it can be rather slow. Not surprisingly, our users would rather not have to wait for these checks, so in [bug 760356](https://bugzilla.mozilla.org/show_bug.cgi?id=760356 "Bug 760356 - Only show the add-on compatibility UI when actually necessary") we implemented a less conservative approach:

- Keep track of when we last did a background add-on update check, so we know how out of date our metadata is.
- On the first run of a new Firefox version, only interrupt startup if the metadata is too out of date (two days, in the current implementation) \*or\* if some add-ons were disabled by this Firefox upgrade but are allowed to be upgraded by the user.

## Did it work?

Yes! On the Aurora channel, we went from interrupting 92.7% of the time on the 30 -&gt; 31 upgradeÂ (378091 out of  
407710 first runs reported to telemetry) to 74.8% of the time (84930 out of 113488) on the 31 -&gt; 32 upgrade, to only interrupting 16.4% (10158 out of 61946) so far on the 32 -&gt; 33 upgrade.

The change took effect over two release cycles; the new implementation was in 32, so the change from “interrupt if there are \*any\* add-ons the user could possibly update” to “interrupt if there is a \*newly disabled\* add-on the user could update” is in effect for the 31 -&gt; 32 upgrade. However, since we didn’t start tracking the metadata update time until 32, the “don’t interrupt if the metadata is fresh” change wasn’t effective until the 32 -&gt; 33 upgrade. I wish I had thought of that at the time; I would have added the code to remember the update time into the telemetry patch that landed in 31.

## Cool, what else did we learn?

On Aurora 33, the distribution of metadata age was:

| Age (days) | Sessions |
|---|---|
| &lt; 1 | 37207 |
| 1 | 9656 |
| 2 | 2538 |
| 3 | 997 |
| 4 | 535 |
| 5 | 319 |
| 6 – 10 | 565 |
| 11 – 15 | 163 |
| 16 – 20 | 94 |
| 21 – 25 | 69 |
| 26 – 30 | 82 |
| 31 – 35 | 50 |
| 36 – 40 | 48 |
| 41 – 45 | 53 |
| 46 – 50 | 6 |

so about 88% of profiles had fresh metadata when they upgraded. The tail is longer than I expected, though it’s not too thick. We could improve this by forcing a metadata ping (or a full add-on background update) when we download a new Firefox version, but we may need to be careful to do it in a way that doesn’t affect usage statistics on the AMO side.

## What about add-on upgrades?

We also started gathering detailed information about how many add-ons are enabled or disabled during various parts of the upgrade process. The measures are all shown as histograms in the telemetry dashboard at http://telemetry.mozilla.org;

<dl><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_XPIDB\_STARTUP\_DISABLED</dt><dd>The number of add-ons (both user-upgradeable and non-upgradeable) disabled during the upgrade because they are not compatible with the new version.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_DISABLED</dt><dd>The number of user-upgradeable add-ons disabled during the upgrade.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_METADATA\_ENABLED</dt><dd>The number of add-ons that changed from disabled to enabled because of metadata updates during the compatibility check.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_METADATA\_DISABLED</dt><dd>The number of add-ons that changed from enabled to disabled because of metadata updates during the compatibility check.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_UPGRADED</dt><dd>The number of add-ons upgraded to a new compatible version during the add-on compatibility check.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_UPGRADE\_DECLINED</dt><dd>The number of add-ons that had upgrades available during the compatibility check, but the user chose not to upgrade.</dd><dt>SIMPLE\_MEASURES\_ADDONMANAGER\_APPUPDATE\_UPGRADE\_FAILED</dt><dd>The number of add-ons that appeared to have upgrades available, but the attempt to install the upgrade failed.</dd></dl>For these values, we got good telemetry data from the Beta 32 upgrade. The counts represent the number of Firefox sessions that reported that number of affected add-ons (e.g. 3170 Telemetry session reports said that 2 add-ons were XPIDB\_DISABLED by the upgrade):

| Add-ons affected | XPIDB DISABLED | APPUPDATE DISABLED | METADATA ENABLED | METADATA DISABLED | UPGRADED | DECLINED | FAILED |
|---|---|---|---|---|---|---|---|
| 0 | 2.6M | 2.6M | Â 2.6M | 2.6M | Â 2.6M | 2.6M | Â 2.6M |
| 1 | 36230 | 7360 | Â 59240 | 14780 | Â 824 | Â 121 | Â 98 |
| 2 | 3170 | 1570 | Â 2 | 703 | Â 5 | Â 1 | Â 0 |
| 3 | 648 | 35 | Â 0 | 43 | Â 1 | Â 0 | Â 0 |
| 4 | 1070 | 14 | Â 1 | 6 | Â 0 | Â 0 | Â 0 |
| 5 | 53 | 20 | Â 0 | 0 | Â 0 | Â 0 | Â 0 |
| 6 | 157 | 194 | Â 0 | 0 | Â 0 | Â 0 | Â 0 |
| 7+ | 55 | 9 | Â 0 | 1 | Â 0 | Â 0 | Â 0 |

The things I find interesting here are:

- The difference between XPIDB disabled and APPUPDATE disabled is (roughly) the add-ons installed in system directories by third party installers. This implies that 80%-ish of add-ons made incompatible by the upgrade are 3rd party installs.
- upgraded + declined + failed is (roughly) the add-ons a user \*could\* update during the browser upgrade, which works out to fewer than one in 2000 browser upgrades having a useful add-on update available. I suspect this is because most add-on updates have already been performed by our regular background update. In any case, to me this implies that further work on updating add-ons during browser upgrade won’t improve our user experience much.