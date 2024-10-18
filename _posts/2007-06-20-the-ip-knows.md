---
id: 15
title: 'What Did the Identity Provider Know?'
date: '2007-06-20T14:18:58-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/2007/06/20/the-ip-knows/'
permalink: /2007/06/20/the-ip-knows/
categories:
    - Identity
---

…And when did it know it?

Kim Cameron [sums up the reasons](http://www.identityblog.com/?p=811) why we need to understand the technical possibilities for how digital identity information can affect privacy; in short, we can’t make good policy if we don’t know how this stuff actually works.

But I want to call out one assertion he (and he’s not the only one) makes:

> Â First,Â part of whatÂ becomes evident isÂ that with browser-based technologies likeÂ Liberty, WS-Federation and OpenID, Â NO collusion is actually necessaryÂ for the identity provider to â€œsee everythingâ€&#157;.

The identity provider most certainly does not “see everything”. The IP sees which RPs you initiate sessions with and, depending on configuration, has some indication of how long those sessions last. Granted, that is \*a lot\* of information, but it’s far from “everything”. The IP must collude with the RPs to get any information about what you did at the RP during the session.