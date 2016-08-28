---
layout: post
title:  "Tuning Active Record Queries"
date:   2016-08-28 14:25:08 -0400
categories: jekyll update
---
When we write Rails application it is easy to be lazy. To simply call `Model.all`
and iterate in the view is tempting, especially if we are in an early phase of
development with little data stored. If this works for you then you may stop
reading here. However, if you find your pages load slowly as your database increases
in size, then re-examining these calls can be a worthwhile step.

Databases are really good at dealing with data. As a result, they know how to
efficiently answer most <i>good</i> questions. If your DB seems slow then you're
probably asking <i>terrible</i> questions. This post is about how to not ask
<i>terrible</i> questions.

<i>Spoiler: use Active Record methods.</i>

<h2>What are <i>good/terrible</i> questions?</h2>
<i>Good</i> questions are <b>specific</b>. They tell the DB <b>exactly</b> what
they want to know, and often the DB has been prepped to receive them.

<i>Terrible</i> questions are <b>vague</b>. The author of such a question acts
as though they don't know what they want, so they often ask for far too much.
<h2>Be More Specific</h2>
<h3>Something is Better than Everything</h3>
For a blog application with a model `Post` having `title`, `author`, and `body`
attributes you can reasonably expect that `body` will frequently be quite large.
By default all attributes are returned. Let's avoid that.

<i>Want only a single attribute from a model?</i> Use `pluck` to ask for it. if you just want titles
`Post.pluck(:title)` performs
{% highlight ruby %}
SELECT posts.title FROM posts
{% endhighlight %}
avoiding the large `body` portion of the post.

<i>Want more attributes?</i> Use `select`. `Post.select(:title, :author)`
{% highlight ruby %}
SELECT posts.title posts.author FROM posts
{% endhighlight %}
also avoiding the large `body` portion of the post.

<h3>Batching</h3>
`find_each`
<h3>Paginate</h3>
will_paginate needs to be at the right part of the query.
<h3>n+1</h3>
`include` is magic.  
https://github.com/flyerhzm/bullet

<h2>Sort Correctly</h2>
`reverse_order` is not the same as `reverse`
<h2>Coach the DB</h2>
<h3>Cache Counters</h3>
`size` is not the same as `length`
<h3>Indexes</h3>
FK's obviously deserve indices, but what other fields could count?
