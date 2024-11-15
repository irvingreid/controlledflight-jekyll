---
title: 'Add-on Manager JSON databases landed!'
date: '2013-08-12T16:00:02-04:00'
author: irving
layout: post
permalink: /2013/08/12/add-on-manager-json-databases-landed/
categories:
    - Mozilla
    - Software
---

Felipe landed the Addon Repository ([**Bug 853389**](https://bugzilla.mozilla.org/show_bug.cgi?id=853389)) changes on Aug 1, so they rode the uplift train and are currently in Firefox Aurora (25). My changes for the XPI Provider ([**Bug 853388**](https://bugzilla.mozilla.org/show_bug.cgi?id=853388)), aside from the Telemetry measurements, landed on Sunday (Aug 11) and are in the latest nightly (Firefox 26). I still need to make some changes to the Telemetry patch and get that through reviews, but the core of the work is finally done.

There are a few follow-up bugs that need to be prioritized and sorted out:

- <span class="summ" id="902950">[ **902950:** <span class="summ_text">Fix test TODO left over from bug 853388</span>](https://bugzilla.mozilla.org/show_bug.cgi?id=902950)</span>
- <span class="summ" id="902956">[ **902956:** <span class="summ_text">Preserve unknown fields in XPI JSON database</span>](https://bugzilla.mozilla.org/show_bug.cgi?id=902956)</span>
- <span class="summ" id="903093">[ **903093:** <span class="summ_text">Ensure that XPI and AddonRepository JSON is completely written before shutdown</span>](https://bugzilla.mozilla.org/show_bug.cgi?id=903093)</span>
- <span class="summ" id="903818">[ **903818:** <span class="summ_text">Handle shutdown during asynchronous load of JSON XPI Database</span>](https://bugzilla.mozilla.org/show_bug.cgi?id=903818)</span>

Many thanks to Felipe for his work on the Addon Repository, and to Blair McBride for reviewing everything.
