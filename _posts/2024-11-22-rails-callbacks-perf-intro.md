---
title: 'Investigating the performance of Rails "Callbacks" module'
author: irving
layout: post
---
I'm not sure about *your* Rails application, but the one I worked on at my previous employer had a
*lot* of callbacks, both for controllers and ActiveRecord models. Over the years I spent a lot of time
looking at stack traces and debugging through those code paths, and Premature Optimization Brain had
me thinking "this must be slow, right?"

The [`ActiveSupport::Callbacks`](https://github.com/rails/rails/blob/v8.0.0/activesupport/lib/active_support/callbacks.rb)
module is [used in a lot of places](https://github.com/search?q=org%3Arails+ActiveSupport%3A%3ACallbacks&type=code).
Messing with it is going to be risky; there's lot of room to break things if we make changes that don't support all the corners.

The other benefit I'm holding out hope for is that we could make the control flow more obvious in stack traces and the debugger.
The `Callbacks` module creates anonymous `Proc`s and does a lot of looping through arrays of objects to decide which ones to
call and when. It's quite difficult to figure out that the code that's running is due to a declaration
like `before_validation :is_object_wiggly` in a parent class,
and that stepping forward proceeds to `before_validation :apply_duck_tape` defined in the current object's subclass...

But first, that thing I said before about "Premature Optimization Brain". How much overhead does the `Callbacks` module add?
I don't know. Until we have some numbers, there's no point making any changes to the implementation. Speeding up a particular
code path by 99% does nothing, if that code only represents 0.1% of overall application execution time. 
