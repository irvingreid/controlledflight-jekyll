---
id: 29
title: 'Nat Pryce has some good ideas about writing test code'
date: '2007-12-26T15:50:07-05:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/2007/12/26/nat-pryce-has-some-good-ideas-about-writing-test-code/'
permalink: /2007/12/26/nat-pryce-has-some-good-ideas-about-writing-test-code/
categories:
    - Software
---

Nat Pryce over at Mistaeks I Hav Made has an interesting series on writing test code, starting with [Test Data Builders](http://nat.truemesh.com/archives/000714.html) and then enhancing the technique with

- [Define common state and avoid aliasing problems](http://nat.truemesh.com/archives/000724.html)
- [Combining builders](http://nat.truemesh.com/archives/000726.html)
- [Emphase the domain model with factory methods](http://nat.truemesh.com/archives/000727.html)
- [Factoring out duplicated logic creates a domain-specific embedded language for testing](http://nat.truemesh.com/archives/000728.html)

This is the kind of stuff that makes me feel good about programming as a craft – the equivalent of having a master carpenter sit down with you and explain how to make a really clean dovetail joint.

Of course, the techniques he describes aren’t just for test code. I enjoy reading and maintaining applications designed with this sort of literate, readable structure.