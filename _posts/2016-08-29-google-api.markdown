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
2. download service account json
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

- GOOGLE_PRIVATE_KEY -> private_key
- GOOGLE_CLIENT_EMAIL -> client_email
- GOOGLE_ACCOUNT_TYPE -> type

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
Once this is done `googleauth` will use these environment variables for the
`get_application_default` message.

[googleauth]: "https://github.com/google/google-auth-library-ruby"
