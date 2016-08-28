---
layout: post
title:  "Tuning Active Record Queries"
date:   2016-08-28 14:25:08 -0400
categories: jekyll update
---
When we write Rails application it is easy to be lazy. To simply call `Model.all`
and iterate in the view is tempting, especially if we are in an early phase of
development with little data stored. If this works for you then you may stop
reading here.
However, if you find your pages load slowly as your database increases
in size, then re-examining these calls can be a worthwhile step.
