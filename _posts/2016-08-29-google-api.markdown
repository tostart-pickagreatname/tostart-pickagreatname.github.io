---
layout: post
title:  "Querying Google Compute with Ruby"
date:   2016-08-29 15:00:00 -0400
categories: gcloud api
---
<i>Note: This is a work in progress...</i>

Recently I queried the Google Compute Engine API and found the authentication setup to be difficult
to navigate. I felt that this process needed a single document example workflow,
which I have chosen to publish here. This document assumes you've already got a
Google Cloud project running.

Steps:

1. add service account
The service account is typically used for communication from a backend server to
the Google Cloud APIs. You can create a new service account and download the
json credential file by following [Google's documentation][service-account].
2. enable the associated API
Different Google Cloud components allow querying via different APIs. To make your
project's data available via an API this API must first be enabled for your project.
In our case, we'll use the Google Compute Engine API. Go to your project's API
dashboard, pick the API you're interested in working with, and enable it.
3. set googleauth creds
There are two methods covered below.  Both use data from the service account
json file along with the `googleauth` gem.  The scope is a list of API endpoints that your authentication is
supposed to be good for. The scopes are [listed by Google][scopes] for quick reference.
4. run your queries
There are two gems I've used to query google. The `gcloud` gem provides more of a
high-level administrative view, whereas the `google-api-client` gem gives more
granular information. Working with the `google-api-client` usually involves
creating an instance of the associated service class and
then running queries as required.

<h2>Using .json File</h2>
Remember that json file you downloaded in step 1? The `googleauth` gem can be pointed
directly at it by setting an environment variable. Then it's
`export GOOGLE_APPLICATION_CREDENTIALS=~/secrets.json`

{% highlight ruby %}
require 'googleauth'

scopes = ['https://www.googleapis.com/auth/cloud-platform',
          'https://www.googleapis.com/auth/compute']
creds = Google::Auth.get_application_default(scopes) #produces a DefaultCredentials method

{% endhighlight %}

<h2>Using Environment Variables</h2>
The `googleauth` gem sets a variety of local variables based on the .json secrets
file. Using an actual secrets file is not always possible in a production deployment.
For example, if you are using a service like Heroku, it is not natural to include
files that aren't kept in your github repo, and you probably don't want to keep
your secrets.json in your github repo. Directly setting environment variables is
typically preferred for this kind of secret information.

There isn't currently an example of this at [Google Auth Github][googleauth].
However, examining the code we notice that `.get_application_default` checks
for the existence of `PRIVATE_KEY_VAR` and `CLIENT_EMAIL_VAR` and then sends the
`make_creds` message. Tracing it back we notice
{% highlight ruby %}
PRIVATE_KEY_VAR = 'GOOGLE_PRIVATE_KEY'.freeze
CLIENT_EMAIL_VAR = 'GOOGLE_CLIENT_EMAIL'.freeze
ACCOUNT_TYPE_VAR = 'GOOGLE_ACCOUNT_TYPE'.freeze
{% endhighlight %}

and these are the three variables required. To set them, we simply open up our
downloaded json file and pull out the variables that are relevant. The association is

- `GOOGLE_PRIVATE_KEY` <-> `private_key:`
- `GOOGLE_CLIENT_EMAIL` <-> `client_email:`
- `GOOGLE_ACCOUNT_TYPE` <-> `type:`

<i>Note: The private_key is a multi-line string. Use of `\n` in this environment
variable will not turn into newlines in rails and will result in `nested asn1 error`.
The solution is to wrap the key with double-quotes and create a multi-line
environment variable like so:</i>
{% highlight ruby %}
export GOOGLE_PRIVATE_KEY="
-----BEGIN PRIVATE KEY-----
therandomstuffthatyouexpect
tobeinyourmultilinekeywithout
theexplicitnewlinecahracters
fejoefjwiof=
-----END PRIVATE KEY-----
"
{% endhighlight %}
Once this is done `googleauth` will use these environment variables for the
`get_application_default` message.

<h2>Query Compute Instances</h2>
Here's a quick and dirty example of how to query the Compute service for a list
of compute instances. We need to first make the Compute Service instance and then
tell it what our form of authorization is, referring to the above generations of
the `creds` variable. Once that's set it's ready to go, so we have:

{% highlight ruby %}
require 'google/apis/compute_v1'
cs = Google::Apis::ComputeV1::ComputeService.new
cs.authorization = creds
project = "your_project_name"
zone = "us-east1-b" # for example, depends on where your instance is.
cs.list_instances(project, zone)
{% endhighlight %}

That's it!  For reference I found [Google's github repo][googleapi] to be the
best source for methods, with fairly readable comments.

[googleauth]: https://github.com/google/google-auth-library-ruby
[service-account]: https://cloud.google.com/storage/docs/authentication#generating-a-private-key
[scopes]: https://developers.google.com/identity/protocols/googlescopes
[googleapi]: https://github.com/google/google-api-ruby-client
