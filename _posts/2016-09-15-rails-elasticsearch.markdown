---
layout: post
title:  "Using Elasticsearch with Rails"
date:   2016-09-16 12:00:00 -0400
categories: elasticsearch rails
---

I wanted to find out what would be required to add elasticsearch to my site for
quick searches of the data that already existed in it. Since elasticsearch has
been around for a little while I decided to start by building out using an available
gem. A quick Google search dropped me into [elasticsearch-rails][esrails], which
seemed like a fairly robust starting point.

My first step was to add elasticsearch to existing models. To do this I used the
`model` portion of the elasticsearch gem.

{% highlight ruby %}
require 'elasticsearch/model'

class SearchText < ApplicationRecord
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
end
{% endhighlight %}

[esrails]: https://github.com/elastic/elasticsearch-rails
