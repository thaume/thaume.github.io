---
layout: post
title: Heroku, Ember-CLI and safe OAuth2 from the browser
date: 2015-03-26
type: post
published: true
tweetId: 581381986809352192
meta:
  seo_title: Heroku, Ember-CLI and safe OAuth2 from the browser
  seo_description: How to effectively and securely use the OAuth2 standard to authorize API authentication calls from the browser with Ember-CLI and ember-simple-auth.
  seo_keywords: emberjs, ember-data, ember-cli, oauth, oauth2, security, ember-simple-auth, simple, auth, heroku
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

This new year started for me with the creation of a [new tech company](http://www.blaaast.co). The first project we had was to build a recruitment platform for a social "Tour de France" of entrepreneurship. It was an awesome project written 100% in Ember + io.js !

During the creation of this web application, I faced a chalenging problem: making OAuth2 calls to an API that does not live on the same domain as my Ember application does. The main problem was not CORS, it works pretty well with modern browsers, but OAuth2 itself.

## The OAuth2 strategy from a client-side app

To make it simple: when you want to use OAuth2, you need to send the `client_id` and the `client_secret` so that the API can recognize which client you are and issue an access token to your user.

The problem is that we are in the browser and you don't want to use a `client_secret` from the browser, waiting to be found.

My first atempt at solving this problem was to add a `client_id` header to my authentication call (`/authentication/token`) :

{% highlight javascript %}
// app/initializers/authentication.js

import Ember from 'ember';
import Session from 'simple-auth/session';
import Authenticators from 'simple-auth-oauth2/authenticators/oauth2';

export default {

  initialize: function (container) {

    ...

    Authenticators.reopen({
      makeRequest: function (url, data) {
        data.client_id = '123412341234';
        return this._super(url, data);
      }
    });
  }
};

{% endhighlight %}

This way I was able to catch the `client_id` and ignore the fact that there's no `client_secret` that comes with it IF the request comes from the right domain (but well, anyone can hack that).

Alright, it works. But it's not standard and it feels 'hacky'.

## Bye bye CORS, hello Nginx proxy

After a couple of days of euphoric development I realised that I had to support IE9. No big deal, except for CORS (CORS can sorta' work on IE8/9 but you need to load an external library).

I realised that it would be far easier to proxy all my requests through an Nginx proxy no matter what is the browser making the requests. Fortunately, the Heroku buildpack comes with an Nginx for free ! You just need to set a couple of environment variables and it Just Works. Alright, all the ajax requests to `http://myappdomain.com/api` are proxied to the API.

Since both the iojs API and the Ember app are hosted at Heroku Europe, they can communicate really fast together through this awesome protocol called HTTP(S). The response time is really great and the measured request timing is almost similar in Nginx's and API's logs.

## Wait, I proxy all my ajax through an Nginx proxy ?

Once everything was set up correctly and working like a charm, it hit me: "I could add a `client_secret` header to the proxied authentication request at the Nginx level to fully secure OAuth2 !". After a quick chat with some cool guys on the ember IRC and positive feedback on the idea, I started working on it.

To make it really secure and private, the `client_secret` has to be set through an environment variable, this way only the admin of the Heroku app can access/edit them (and obviously the Nginx proxy has access to it too).

So the Nginx proxy now adds the `client_secret` to the `/authentication/token` request. It just feels right to have both the `client_id` and the `client_secret` sent to the API for authentication.
