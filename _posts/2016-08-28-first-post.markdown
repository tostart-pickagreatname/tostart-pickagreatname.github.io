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
You want to use `reverse_order` instead of `reverse`. `reverse` converts the ActiveRecord::Relation
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

<h3>Include Everything You Need</h3>
Suppose each `Post` `has_one` `author` via `author_id` with an index page for the
posts which displays each post and its author's name via the `name` attribute of
`Author`.  Suppose you do this with `@posts = Post.all` in the controller and
the following in the view
{% highlight ruby %}
@posts.each do |post|
  <%= post.title %>, by <%= post.author.name %>
end
{% endhighlight %}
The result is that for each `post`, a call is made to the `authors` table of the
sort
{% highlight ruby %}
SELECT authors.* FROM authors WHERE author.id == #{post.author_id}
{% endhighlight %}
As a result we have 1 initial query to populate `@posts`, and then for each `post`
we have an additional query to acquire its author. If there are <i>n</i> rows in
the `posts` table, then there will be <i>n+1</i> queries to the database to load
this index, slowing down the rendering of the page. How can we avoid this?
Set `@posts = Post.include(:author)`. This tells Active Record to perform a query
to the `authors` table to get the related authors, thus turning those `n` queries
into a single query. Specifically, it will perform
{% highlight ruby %}
SELECT authors.* FROM authors WHERE author.id IN (<list of ids from posts table>)
{% endhighlight %}

Not sure if you're doing this somewhere? There are a few gems to help you look
for this problem. I recommend the [bullet gem][bullet].

<h3>Batching</h3>
Suppose processing your query would be fast, but it's taking up too much memory?
If there are a large number of rows in the table, we may use `find_each` to break
the query up into batches to reduce memory usage. The default batch size is 1000. If we want to perform
a block against every post using a method of `update_post`, we could do this by
{% highlight ruby %}
Post.find_each(batch_size: 10) do |post|
  post.update_post
end
{% endhighlight %}
The result would be a series of queries for 10 records apiece, rather than all at
once.

<h2>Remarks</h2>
Most of the above was discovered by following a 2-step process.

- I found a page that rendered slowly.
- I ran the queries in rails console and counted/examined the SQL queries.
However, many of the efficiencies came only <i>after</i> I knew what I was going
to use the data for. It's important not to get too hung up on this at an early stage.


[bullet]: https://github.com/flyerhzm/bullet
