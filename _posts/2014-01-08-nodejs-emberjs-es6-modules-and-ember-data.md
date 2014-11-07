---
layout: post
title: NodeJS, EmberJS, ES6 modules and Ember-Data
date: 2014-01-08
type: post
published: true
meta:
  seo_title: EmberJS, ES6 modules and Ember-Data
  seo_description: Putting the ‘mailbox' example of the emberjs.com website
    on steroïds with Ember-Data and ES6 modules and query data from NodeJS
  seo_keywords: 'Ember, Ember js, EmberJS, ES6 modules, Ember modules, Ember-Data,
    ember data, express, '
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

> **Update 28/06/2014**
> Ember App Kit is now deprecated in favor of [Ember-CLI](http://www.ember-cli.com/)

In this blog post about EmberJS and Ember-Data, we'll use the example from the [last blog post series](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-1/) (you can find it [in this git repo](https://github.com/thaume/emberjs-es6-modules) on the branch master) and we'll plug in the RESTAdapter in order to get data from an Express server instead of using fixtures. Clone the git repo, and follow me !

![tomster-under-construction](/assets/img/tomster-under-construction.png)

## Getting the RESTAdapter to work

The Ember App Kit has a very useful little feature called 'API-stub', it let you create routes on Express and respond with data. We will use this API-stub to create our JSON that we want to send to our Ember app through Ember-Data, so let's configure our ApplicationAdapter, you will find here : <code class="language-markup">app/adapters/application.js</code>

{% highlight javascript %}
var ApplicationAdapter = DS.RESTAdapter.extend({
    namespace: 'api'
});

export default ApplicationAdapter;
{% endhighlight %}

As simple as that, our RESTAdapter is now configured to work with our Express server.

### Setup the API response in NodeJS

The Express server also needs to be configured so that it can talk with our Ember-App. We will send a JSON when it gets a GET request on <code class="language-markup">localhost:8000/api/mailboxes</code> (Ember-Data does automatic pluralization on ressources name),  the Express routes handler file is here <code class="language-markup">api-stub/routes.js</code> and here is what you need to do :

{% highlight javascript %}
module.exports = function(server) {

  // Create an API namespace, so that the root does not
  // have to be repeated for each end point.
  server.namespace('/api', function() {

    // Return fixture data for '/api/mailboxes'
    server.get('/mailboxes/', function(req, res) {
      var data = { ... }
      res.send(data);
    });

  });

};
{% endhighlight %}

The JSON that we will send back is just the JSON Fixtures we used with the FixturesAdapter but it needs a little bit of rework to work with Ember-Data.

## Formatting/normalizing your data

Ember-Data expects a particular type of data format, you need to normalize your data and use 'foreign keys' the same way you would with a relational database. Our JSON fixtures looked like that :

{% highlight javascript %}
{
  "name": "Inbox",
  "id": "inbox",
  "messages": [
    {
      "id": 1,
      "subject": "Welcome to Ember",
      "from": "email",
      "to": "email",
      "date": new Date(),
      "body": "Welcome to Ember. We hope you enjoy your stay"
    }, {
      "id": 2,
      "subject": "Great Ember Resources",
      "from": "email",
      "to": "email",
      "date": new Date(),
      "body": "Have you seen embercasts.co? How about emberaddons.co?"
    }
  ]
}, {
  "name": "Spam",
  "id": "spam",
  "messages": [
    {
      "id": 3,
      "subject": "You have one the lottery!!!111ONEONE",
      "from": "email",
      "to": "email",
      "date": new Date(),
      "body": "You have ONE the lottery! You only have to send us a small amount of monies to claim your prize"
    }
  ]
}, {
  "name": "Sent Mail",
  "id": "sent-mail",
  "messages": [
    {
      "id": 4,
      "subject": "Should I use Ember",
      "from": "email",
      "to": "email",
      "date": new Date(),
      "body": "Ember looks pretty good, should I use it?"
    }
  ]
}
{% endhighlight %}

In order to be Ember-Data ready, we will have to move our messages into another object called 'messages' and give them an ID to inject them back in our mailboxes, it goes like this :

{% highlight javascript %}
"messages": [{
  "id": 1,
  "subject": "Welcome to Ember",
  "from": "email",
  "to": "email",
  "date": new Date(),
  "body": "Welcome to Ember. We hope you enjoy your stay"
}, {
  "id": 2,
  "subject": "Great Ember Resources",
  "from": "email",
  "to": "email",
  "date": new Date(),
  "body": "Have you seen embercasts.com? How about emberaddons.com?"
}, {
  "id": 3,
  "subject": "You have one the lottery!!!111ONEONE",
  "from": "email",
  "to": "email",
  "date": new Date(),
  "body": "You have ONE the lottery! You only have to send us a small amount of monies to claim your prize"
}, {
  "id": 4,
  "subject": "Should I use Ember",
  "from": "email",
  "to": "email",
  "date": new Date(),
  "body": "Ember looks pretty good, should I use it?"
}]
{% endhighlight %}

The mailbox part will look like that :

{% highlight javascript %}
"mailbox": [{
  "name": "Inbox",
  "id": "inbox",
  "messages": ["1", "2"]
}, {
  "name": "Spam",
  "id": "spam",
  "messages": ["3"]
}, {
  "name": "Sent Mail",
  "id": "sent-mail",
  "messages": ["4"]
}]
{% endhighlight %}

As you can see, the mailbox JSON has a 'messages' attribute that is an array filled with numbers. Those numbers are the 'foreign keys' that will enable us to attach referenced messages into each mailbox. If a mailbox has a 'messages' attribute equal to <code class="language-javascript">["1", "2"]</code> then the messages with IDs "1" and "2" will be attached to this mailbox once the data reaches Ember. We can now merge those two JSON into <code class="language-javascript">var data</code> and send it over to Ember-Data once an ajax call is made to our Express server on the mailboxes route.

## Creating the Ember Models

We need to define two models, one for each ressource ('message' and 'mailbox') in order to define their structures.

### Message model

The message model will be pretty straightforward, we will just copy/paste the JSON attributes and set them as DS.attr(). We could define a type for each attributes like <code class="language-javascript">DS.attr('string')</code> or <code class="language-javascript">DS.attr('date')</code> but for the purpose of this example, it is unecessary.

{% highlight javascript %}
var attr = DS.attr;

var Message = DS.Model.extend({
  number: attr(),
  subject: attr(),
  from: attr(),
  to: attr(),
  date: attr(),
  body: attr()
});

Message.idField = 'number';

export default Message;
{% endhighlight %}

As you can see, the "id" attribute became "number", this is due to the fact that ember will not let you name a model's attribute ID. Instead you will need to create an idField for the model and set the attributes that should be the ID. In this example, "number" is the "id" of the model.

### Mailbox model

The mailbox model will be slightly trickier since we need to define a 'relationship' with the message model. The 'mailbox' can have one or more 'messages' so the relationship will be <code class="language-javascript">hasMany</code> 'message'. It looks like that :

{% highlight javascript %}
var attr = DS.attr,
    hasMany = DS.hasMany;

var Mailbox = DS.Model.extend({
  number: attr(),
  name: attr(),
  messages: hasMany('message')
});

Mailbox.idField = 'number';

export default Mailbox;
{% endhighlight %}

This <code class="language-javascript">hasMany</code> relationship tells Ember-Data that when a mailbox model is loaded, it also needs to fetch the messages for which the ID is inside the messages array and then populate this messages array with the actual messages.

## The Application route

Now that our model is able to fetch data from our Express server, we need to inject this data inside the views. We will use the 'model' hook of the ApplicationRoute to do that :

{% highlight javascript %}
var ApplicationRoute = Em.Route.extend({
  model: function() {
    return this.store.find('mailbox');
  }
});

export default ApplicationRoute;
{% endhighlight %}

We ask Ember-Data throught the <code class="language-javascript">store</code> object to find all the mailbox objects. Under the hood, Ember will make an ajax call to the route <code class="language-markup">localhost:8000/api/mailboxes</code> and get a JSON from this URL. You do not need to import models into your ApplicationRoute module, Ember will automatically find models in the folder model when you look for them with the store. Once again, conventions make the whole process a breeze.

The last thing you need to do now is to run <code>grunt server</code> in your terminal, and see if everything is working !

You can find the code used in this article in [this git repo](https://github.com/thaume/emberjs-es6-modules/tree/ember-es6-ember-data) on the branch 'ember-es6-ember-data'.
