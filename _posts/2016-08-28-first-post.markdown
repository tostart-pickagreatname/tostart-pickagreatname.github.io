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

<h2>Be More Specific</h2>
`select`
<h2>Paginate</h2>
will_paginate needs to be at the right part of the query.
<h2>Reverse Right</h2>
`reverse_order` is not the same as `reverse`
<h2>n+1</h2>
`include` is magic.  
https://github.com/flyerhzm/bullet
<h2>Cache Counters</h2>
`size` is not the same as `length`
<h2>Indexes</h2>
FK's obviously deserve indices, but what other fields could count?
<h2>Batching</h2>
`find_each`
<h2>Push work to the database</h2>
Instead of loading everything and then creating summaries, try using things that build this data into the database itself and then call it.
