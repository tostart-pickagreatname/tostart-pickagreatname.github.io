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
`model` portion of the elasticsearch gem.  The most compelling example of the
model is likely [here][modelexample].

I first created the following model, and configured my default client

{% highlight ruby %}
require 'elasticsearch/model'

class SearchText < ApplicationRecord
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
end
{% endhighlight %}

{% highlight ruby %}
Elasticsearch::Model.client = Elasticsearch::Client.new host: '127.0.0.1:9200'
{% endhighlight %}

I then created the test script:

{% highlight ruby %}
SearchText.create(text: 'first_text')
SearchText.create(text: 'second_text')
SearchText.create(text: 'third_text')

client = Elasticsearch::Model.client
client.indices.delete index: 'search_texts' rescue nil
client.indices.create index: 'search_texts' rescue nil
client.bulk index: 'search_texts',
            type:  'search_text',
            body:  SearchText.all.as_json.map { |a| { index: { _id: a.delete('id'), data: a } } },
            refresh: true
puts SearchText.search('*').map(&:text)
{% endhighlight %}

The output of this script is predictable:

{% highlight ruby %}
first_text
second_text
third_text
{% endhighlight %}

What do we learn from this? We learn that we need to create the indices. We also
learn that we can manually tell elasticsearch to bulk update using current
contents of the DB.

[esrails]: https://github.com/elastic/elasticsearch-rails
[modelexample]: https://github.com/elastic/elasticsearch-rails/blob/a714d5931182501531d2ce6b767bcb3c97ce56a7/elasticsearch-model/examples/activerecord_associations.rb
