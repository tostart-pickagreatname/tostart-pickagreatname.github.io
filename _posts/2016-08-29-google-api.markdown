---
layout: post
title:  "Querying Google Compute with Ruby"
date:   2016-08-29 15:00:00 -0400
categories: gcloud api
---
<i>work in progress...</i>

Recently I queried the Google Compute API and found the auth. setup to be difficult
to navigate. I felt that this process needed a single document example workflow,
which I have chosen to publish here.

<h2>Using .json File</h2>
Steps:

1. add service account
2. download json
3. pick scope
4. set googleauth creds
5. create compute service instance
6. query the instances

`export GOOGLE_APPLICATION_CREDENTIALS=~/secrets.json`

Then do include things that you care about.
{% highlight ruby %}
require 'googleauth'

scopes = ['https://www.googleapis.com/auth/cloud-platform',
          'https://www.googleapis.com/auth/compute']
creds = Google::Auth.get_application_default(scopes) #produces a DefaultCredentials method

{% endhighlight %}

<h3>query compute instances</h3>

{% highlight ruby %}
require 'google/apis/compute_v1'
cs = Google::Apis::ComputeV1::ComputeService.new
cs.authorization = creds
project = "your_project_name"
zone = "us-east1-b" # for example, depends on where your instance is.
cs.list_instances(project, zone)
{% endhighlight %}

<h2>Using Environment Variables</h2>
