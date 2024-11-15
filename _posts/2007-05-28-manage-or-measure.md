---
title: 'Manage or Measure?'
date: '2007-05-28T23:46:22-04:00'
author: irving
layout: post
guid: 'http://www.controlledflight.ca/2007/05/28/manage-or-measure/'
permalink: /2007/05/28/manage-or-measure/
categories:
    - Business
---

One of the rants that has been bubbling in my head for a while is about the current obsession with “metrics” in management circles. I usually blame this on MBA culture, though I must admit that I don’t have direct evidence to blame the business schools.

The oft-heard mantra is “If you can’t measure it, you can’t manage it.” I think this is missing a first, critical step:

If you don’t understand it, you can’t measure it.

Of course, you can *try* to measure something you don’t understand, but chances are (a) you don’t measure what you think you are measuring, (b) your attempt at measurement alters the system you’re trying to measure, and (c) the result of your measurement is too noisy to draw valid conclusions.

Joel Spolsky’s recent article [“Typical Econ 101 Management”](http://www.joelonsoftware.com/items/2007/05/10.html) led me to re-read his excellent articles on management techniques: [  
The Command and Control Management Method](http://http://www.joelonsoftware.com/items/2006/08/08.html "The Command and Control Management Method"),  
[The Econ 101 Management Method](http://www.joelonsoftware.com/items/2006/08/09.html), and [The Identity Management Method](http://www.joelonsoftware.com/items/2006/08/10.html). What Joel calls Econ 101 Management is exactly the sort of poor measurement I’m referring to, applied to the context of getting individuals to do The Man’s bidding.

The Previous Employer demonstrated a couple of interesting variants on the measurement problem. The first one came in the form of “industry benchmarks”. We looked at what a bunch of other companies spent on various business functions, at a macroscopic level. For example, we spent X% of revenue on IT, while “competitors” spent X’%; we spent Y% on R&amp;D, versus Y’%. Every place where our spend was higher than our competitors was taken as an opportunity for wide scale budget cuts.

Now, I think it’s a *great* idea to look around for good ideas on how to run your business. The problem comes in when you turn that comparison-with-a-competitor into a metric that applies to one specific component of your business. Unless your middle managers have both great backbones and great understanding of the overall business, they’re going to break a *lot* of things to make that metric turn green on your executive dashboard. Next year you’re going to discover that you’re lagging the competitor on some other benchmark (like employee morale…) and you’ll have to invent another metric to try and manage that, and your managers will break other things to turn the new metric green, and you’ll wonder why all the productive employees are leaving.

The other object lesson from the Previous Employer was around the software development life cycle and defect tracking. Automated processes scrape the defect tracking database for all products and produce various statistics, such as the rate of new defect arrivals, time to address open defects, level of customer impact, etc. The best part of these statistics is that they were used to apply competing metrics to different groups: Developers need to keep the defect arrival rate low, or they’re not allowed to ship to customers (because a high defect arrival rate indicates unstable code). At the same time, the Product Test group needs to file a certain number of defects per tester per week (because a low rate indicates they’re ineffective). This led to an unhealthy divide between developers and testers as we fought over when and how to track defects, with each side trying to game the process so that they wouldn’t look bad on the monthly management charts.

And yet, when it comes right down to it, I’m a big fan of the Scientific Method. That means I *do* want to measure the effectiveness of both broad scale and fine grain activities; how else can we tell whether we’re improving? The key is the *understanding*, all the way down the management hierarchy. A measurement isn’t a business outcome. You should *never* ask your organization to change a measurement (either explicitly, or implicitly by rewarding based on the measurement). Always ask people to change the business outcome, explain why you think the measurement is a reasonable approximation of the business outcome, and watch very carefully to make sure that decisions are directed toward the business outcome, not the measurement. And in the spirit of Continuous Improvement, constantly review your metrics to make sure they really are driving toward better business outcomes.
