---
title: 'Add-on compatibility checks and new Firefox versions'
date: '2014-01-07T12:42:20-05:00'
author: irving
layout: post
permalink: /2014/01/07/add-on-compatibility-checks-and-new-firefox-versions/
categories:
    - Mozilla
tags:
    - irving
---

I’ve been working on [**Bug 772484**](https://bugzilla.mozilla.org/show_bug.cgi?id=772484 "Bug 772484 - extension check dialog is annoying and can effectively hang the Firefox process") – “extension check dialog is annoying and can effectively hang the Firefox process”, and Vladan suggested I post an outline of the issues and proposed changes.

The underlying problem is that when you start up an updated version of Firefox for the first time, we want to make sure that as many as possible of your add-ons keep working, but that we disable any add-ons that are known to cause problems with the new Firefox. This process could include updating add-ons when the user has an old version of the add-on installed and a newer version would be compatible with the new Firefox.

If we don’t do this check, there are two possible bad outcomes: We start up with an add-on disabled when we could have enabled it, which leaves the user without their desired Firefox configuration, or we start up with an add-on enabled that is not compatible, breaking some or all of the browser. When we switched to ‘compatible by default’ for add-ons, to make the rapid release process easier for add-on authors and users, we were worried that we might need to quickly publish compatibility overrides to protect users from cases where we allowed an add-on to load even though the new browser was not compatible with it. To give us the best chance of not breaking the user’s browser, the current implementation completely blocks the browser start up until after we get updated compatibility information from addons.mozilla.org (AMO).

The compatibility check is triggered if the software version of the browser is different from a saved preference containing the version ID of the last run. If the versions are different, the add-on manager startup goes like:

1. Check all installed add-ons to see if they are compatible with the new browser version, keeping track of any that are enabled or disabled.
2. Put up a modal dialog box that blocks Firefox start up, and prevent the user from cancelling out of the dialog.
3. Send a search request (the “metadata ping”) to AMO with a list of all installed add-on IDs. AMO will return up-to-date add-on metadata for any add-ons it knows, including compatibility overrides for add-ons not hosted on AMO but where Mozilla is aware of compatibility issues.
4. Update stored add-on compatibility for all add-ons, enabling or disabling any for which the AMO information is different from what we knew before.
5. At this point we allow the user to exit the dialog box.
6. For all installed add-ons, request add-on update information directly from the add-on’s update URL, which could be AMO again or could be an external add-on hosting location.
7. Update the stored compatibility info again based on any new info from step 6.
8. Display a list of all the add-ons that used to be compatible, but aren’t any more, and ask the user whether we should check whether new versions of these add-ons are available.
9. If the user chooses to proceed, request the add-on’s update URL again ([**Bug 960597**](https://bugzilla.mozilla.org/show_bug.cgi?id=960597 "Bug 960597 - Extension check UI fetches add-on update info twice")) to find out whether there is a different version of the add-on that is compatible with the new browser version.
10. Present a list of add-ons for which a compatible version is available, and allow the user to select which ones to download and update.
11. Download and apply the selected add-ons.
12. Continue starting up the browser, including loading enabled add-ons.

Everything up to step 11 happens early enough in the start up process that no add-ons have been loaded yet; this allows us to apply any necessary add-on updates without restarting the browser.

[**Bug 772484**](https://bugzilla.mozilla.org/show_bug.cgi?id=772484 "Bug 772484 - extension check dialog is annoying and can effectively hang the Firefox process") is mostly concerned with steps 2 through 4, where the modal dialog does not allow the user to cancel. If the browser is on a disconnected network where packets sent to Internet hosts are silently discarded, we may wait for either a DNS or TCP connection timeout (up to 70-75 seconds) before we allow the user to interact with the browser.

The metadata ping to AMO in step 3 has a cancel() API, but attempting to use it uncovered [**Bug 966374**](https://bugzilla.mozilla.org/show_bug.cgi?id=966374 "Bug 966374 - Race condition in AddonRepository.cancelSearch()"). We could make this search interruptible, in which case the effect would be the same as if the search failed – we would start the browser with out-of-date compatibility information, which could lead to running with an incompatible add-on enabled. If we leave the search running in the background, it is almost certain that the browser will proceed to initialize add-ons based on out of date information before the search completes; in this case the browser would enable/disable restartless add-ons on the fly, and mark those that require restart so that they would be in the right state on next startup. In the mean time it is still possible to be running with incompatible add-ons. If we want to ask the user to restart, we would need new UI.

It was possible to close the dialog during the update requests in step 6, but some of the background work was not being cleaned up. Support was added in [**Bug 925389** ](https://bugzilla.mozilla.org/show_bug.cgi?id=925389 "Add-on update check is not cancelled when add-on is uninstalled")so that we can now cancel those requests if we want. As with step 3 above, if we allow the user to cancel during this step we may start with incompatible add-ons, and if we allow the requests to complete in the background we may need a restart to get into a consistent state.

I have work in progress that allows the user to close the add-on check dialog during the initial AMO search (steps 3 and 4) but lets the search complete and updates the add-on database with new compatibility information when it’s done. If the user cancels during the first group of update URL requests (step 6), any requests that have not completed are cancelled. This would in rare cases lead to starting up with incompatible add-ons (or without compatible ones), but I think it’s a reasonable trade off – particularly since a likely common reason the user wants to cancel is that the requests are taking too long, and cancelling them leads to the same result as letting them time out.

My WIP patch is also adding tests for closing the add-on update dialog at various other steps where it is possible, which has uncovered a couple of other places where we weren’t cleaning up.

In the mean time, Blair has been working on [**Bug 760356**](https://bugzilla.mozilla.org/show_bug.cgi?id=760356 "Bug 760356 - Only show the add-on compatibility UI when actually necessary") which reworks the add-on check to avoid blocking startup whenever possible. See the WIP patches on that bug to understand the fine details of the change, but the rough idea is:

- Keep track of when we last updated add-on compatibility information (typically once per day, when we do the automatic blocklist and add-on background update check)
- At start up, skip the compatibility dialog if any of the following are true: 
    - No add-ons can be updated by the user (e.g. all were pre-installed with the application binaries rather than in the user’s profile)
    - No add-ons were made incompatible and disabled by the browser upgrade
    - the last update of compatibility information was recent enough \*and\* none of the newly-incompatible add-ons can be updated by the user
- If we do show the compatibility dialog, it doesn’t try to update all add-ons, only the ones disabled by the browser upgrade.

I’m open to suggestions about both of these approaches, either here or on the relevant bugs.
