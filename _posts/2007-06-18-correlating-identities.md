---
id: 14
title: 'Correlating Identities over Time'
date: '2007-06-18T22:53:19-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/2007/06/18/correlating-identities/'
permalink: /2007/06/18/correlating-identities/
categories:
    - Identity
---

[Kim Cameron](http://www.identityblog.com/?p=806) responds to [Paul Madsen](http://connectid.blogspot.com/2007/06/colluding-with-yourself.html) responding to Kim Cameron, and I wonder what it is about Canadians and identity…

Paul points out that Kim is missing one possible form of collusion, in which a single site correlates repeated visits from an individual even though they don’t know that individual’s name. Kim, in response, asks:

> But I have to admit that I have not personally been that interested in the use case of presenting "managed assertions"; to amnesiac web sites. In other words, I think the cases where you would want a managed identity provider for completely amnesiac interactions are fairly few and far between. (If someone wants to turn me around me in this regard I'm wide open.)

It turns out that [Shibboleth](http://shibboleth.internet2.edu/), in particular, has a very clear requirement for this use case. [FERPA](http://www.ed.gov/policy/gen/guid/fpco/ferpa/index.html) requires that educational institutions disclose the least possible information about students, staff and faculty to their partners. The example I heard, back in the early days of SAML, was of an institution that had a contract with an on-line case law research provider such that anyone affiliated with the law school at that institution could look up cases.

In this case, the “managed identity provider” (representing the educational institution) needs to assert that the person visiting right now is affiliated with the law school. However, the provider has no need to know anything more than that, and therefore the institution has a responsibility under FERPA to not give the provider any extra information. “The person looking up Case X right now is the same person who looked up Case Y last week” is one of the pieces of information the institution shouldn’t share with the provider.
