# Twitter demo app

## Preliminaries

As follows is described the software required to run Python apps against Twitter API.

1. Create a virtualenv
	* To create local virtualenv, `virtualenv <dir>` e.g. `virtualenv venv`
	* Activate the new environment, `source <dir>/bin/activate` e.g. 
          `source venv/bin/activate`
2. Open your twitter account. Visit the following link [twitter apps](https://apps.twitter.com/)
3. Then create a new app 
4. When you successfully created your twitter app, look for the following itmes in the "Kyes and Access Tokens" tab
	* Consumer Key (API Key)
	* Consumer Secret (API Secret)
5. Copy and paste the following code. This is originally found at [link][1]. In this github repository that code can be download from [here](https://github.com/josanabr/mytwitter/blob/master/helper.py) 

~~~~
import urlparse
import oauth2 as oauth

consumer_key = 'my_key_from_twitter'
consumer_secret = 'my_secret_from_twitter'

request_token_url = 'https://api.twitter.com/oauth/request_token'
access_token_url = 'https://api.twitter.com/oauth/access_token'
authorize_url = 'https://api.twitter.com/oauth/authorize'

consumer = oauth.Consumer(consumer_key, consumer_secret)
client = oauth.Client(consumer)

# Step 1: Get a request token. This is a temporary token that is used for 
# having the user authorize an access token and to sign the request to obtain 
# said access token.

resp, content = client.request(request_token_url, "GET")
if resp['status'] != '200':
    raise Exception("Invalid response %s." % resp['status'])

request_token = dict(urlparse.parse_qsl(content))

print "Request Token:"
print "    - oauth_token        = %s" % request_token['oauth_token']
print "    - oauth_token_secret = %s" % request_token['oauth_token_secret']
print 

# Step 2: Redirect to the provider. Since this is a CLI script we do not 
# redirect. In a web application you would redirect the user to the URL
# below.

print "Go to the following link in your browser:"
print "%s?oauth_token=%s" % (authorize_url, request_token['oauth_token'])
print 

# After the user has granted access to you, the consumer, the provider will
# redirect you to whatever URL you have told them to redirect to. You can 
# usually define this in the oauth_callback argument as well.
accepted = 'n'
while accepted.lower() == 'n':
    accepted = raw_input('Have you authorized me? (y/n) ')
oauth_verifier = raw_input('What is the PIN? ')

# Step 3: Once the consumer has redirected the user back to the oauth_callback
# URL you can request the access token the user has approved. You use the 
# request token to sign this request. After this is done you throw away the
# request token and use the access token returned. You should store this 
# access token somewhere safe, like a database, for future use.
token = oauth.Token(request_token['oauth_token'],
    request_token['oauth_token_secret'])
token.set_verifier(oauth_verifier)
client = oauth.Client(consumer, token)

resp, content = client.request(access_token_url, "POST")
access_token = dict(urlparse.parse_qsl(content))

print "Access Token:"
print "    - oauth_token        = %s" % access_token['oauth_token']
print "    - oauth_token_secret = %s" % access_token['oauth_token_secret']
print
print "You may now access protected resources using the access tokens above." 
print
~~~~

6. Modify variables `consumer_key = 'my_key_from_twitter'` and `consumer_secret = 'my_secret_from_twitter'` with your own values

7. Run the program and follow the given steps.

8. Save `oauth_token` and `oauth_token_secret` values.

9. Visit the web page of your recently created twitter app, click on "Keys and Access Tokens" tab and click on button "Test OAuth".

10. Jot down the the following fields
	* Consumer key
	* Consumer secret
	* Access token
	* Access token secret

## Your first twitter client

The following code allows you to retrieve your twitter timeline, [demo-client.py](https://github.com/josanabr/mytwitter/blob/master/demo-client.py)

~~~~

#!/usr/bin/env python
# -*- coding: latin-1 -*-
import oauth2

def oauth_req(url, key, secret, http_method="GET", post_body="", http_headers=None):
    consumer = oauth2.Consumer(key="CONSUMER_KEY", secret="CONSUMER_SECRET")
    token = oauth2.Token(key=key, secret=secret)
    client = oauth2.Client(consumer, token)
    resp, content = client.request( url, method=http_method, body=post_body, headers=http_headers )
    return content
 
home_timeline = oauth_req( 'https://api.twitter.com/1.1/statuses/home_timeline.json', 'YOUR_ACCESS_TOKEN', 'YOUR_ACCESS_TOKEN_SECRET' )

f = open("twittertimeline.txt","w")
f.write(home_timeline)
f.close()

~~~~

Before to run it, replace the following fields in the code:

* `CONSUMER_KEY`
* `CONSUMER_SECRET`
* `YOUR_ACCESS_TOKEN` 
* `YOU_ACCESS_TOKEN_SECRET`

By your own values, respectively:

* Consumer key
* Consumer secret 
* Access token
* Access token secret

## Presenting human-readable results

The following code allows you to read the results of your previous program in a more readable way, [jsonhelper.py](https://github.com/josanabr/mytwitter/blob/master/jsonhelper.py).

~~~~

#!/usr/bin/env python
import json
from pprint import pprint

with open('twittertimeline.txt') as data_file:    
    data = json.load(data_file)

pprint(data)

~~~~

[1]: https://github.com/joestump/python-oauth2/wiki/Twitter-Three-legged-OAuth  "Twitter Three legged OAuth"
