---
layout: post
title:  "Faster Active Record Queries"
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
attributes you can reasonably expect that `body` will be large.
Active Record will return all attributes by default, so let's avoid that.

<i>Want only a single attribute from a model?</i> Use `pluck` to ask for it. If
you just want a list of titles: `Post.pluck(:title)` performs
{% highlight ruby %}
SELECT posts.title FROM posts
{% endhighlight %}
avoiding the large `body` portion of the post.

<i>Want more attributes?</i> Use `select`: `Post.select(:title, :author)` executes
{% highlight ruby %}
SELECT posts.title posts.author FROM posts
{% endhighlight %}
also avoiding the large `body` portion of the post.

<h3>`reverse` is not `reverse_order`</h3>
TL;DR: This is a simple gotcha: `Post.all.reverse` performs a `SELECT *` and then reverses
the , whereas `Post.all.reverse_order` includes an `ORDER BY` statement.

You want to use `reverse_order`. `reverse` converts the ActiveRecord::Relation
into a list. That is, it executes the SQL query and then reverses the returned
values. If you only want a subset of that list you will be returning much more
than you need. For example, suppose you are paginating with a gem like `will_paginate`.
`will_paginate` will try to limit a query to the number of rows it is paginating.
Indeed, if you do something like
{% highlight ruby %}
Post.all.reverse.paginate(page: 1, per_page: 30)
{% endhighlight %}
then the SQL query will pull <b>all</b> the records down and paginate the list.
HOWEVER, if instead you do
{% highlight ruby %}
Post.reverse_order.paginate(page: 1, per_page: 30)
{% endhighlight %}
then your SQL query will return at most 30 rows, which will scale much more
efficiently.


<h3>Batching</h3>
`find_each`
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
