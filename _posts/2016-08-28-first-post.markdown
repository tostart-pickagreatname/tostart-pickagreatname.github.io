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

This blog post contains some useful methods for dealing with common scenarios.

<h3>Something is Better than Everything</h3>
For a blog application with a model `Post` having `title`, `author`, and `body`
attributes you can reasonably expect that `body` will be large.
Active Record will return all attributes by default, so let's avoid that.

<i>Want only a single attribute from a model?</i> Use `pluck` to ask for it. If
you just want a list of titles: `Post.pluck(:title)` performs
{% highlight ruby %}
SELECT posts.title FROM posts
{% endhighlight %}
avoiding the large `body` portion of the post. It is important to note that pluck
is really most useful for
display and aggregation, as it loses structure and retrieves
a simple list of values.

<i>Want more attributes or need the Active Record objects?</i> Use `select`: `Post.select(:title, :author)` executes
{% highlight ruby %}
SELECT posts.title posts.author FROM posts
{% endhighlight %}
also avoiding the large `body` portion of the post.

Try to avoid using these too early in the development process, or you might find
yourself being forced to re-add attributes to these calls.

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

<h3>Sort Data in the DB</h3>
Where possible, we should use the database to sort things. Using the `order` method
is fairly common, but recently I made the mistake of using `reverse` instead of
`reverse_order` and I think it's worth explaining what happened here.
Simply put: you want to use `reverse_order` instead of `reverse`. `reverse` converts the ActiveRecord::Relation
into a list and then sorts it. That is, it executes the SQL query and then reverses the returned
values. This might not have been an issue when it was first implemented, but if it's chained
with other Active Record methods you might get surprising behavior. For example,
suppose you are paginating with a gem like `will_paginate`.
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
Suppose processing your query would be fast, but there are a lot of rows, so it's
taking up a lot of memory. Then we may use `find_each` to break
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

1. I found a page that rendered slowly.
2. I ran the queries in rails console and counted/examined the SQL queries.
However, many of the efficiencies came only <i>after</i> I knew what I was going
to use the data for. It's important not to get too hung up on this at an early stage.

Rails console gives us some great information to go on, like how long it takes
our database to process the query. There is a lot more to this than I could cover
in this blog, but I hope that these ideas are helpful for readers. In general:

- <i>Good</i> queries are <b>specific</b>. They tell the DB <b>exactly</b> what
they want to know and leverage Active Record methods to do so.
- <i>Terrible</i> queries are <b>vague</b>. The author of such a question acts
as though they don't know what they want, so they often ask for too much, or don't
include data that needs to be used later.

[bullet]: https://github.com/flyerhzm/bullet
