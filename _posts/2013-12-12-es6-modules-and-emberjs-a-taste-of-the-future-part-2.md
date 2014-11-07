---
layout: post
title: 'ES6 modules and EmberJS : a taste of the future (part 2)'
date: 2013-12-12
type: post
published: true
meta:
  seo_title: 'ES6 modules and EmberJS : a taste of the future (part 2)'
  seo_description: Second part of the two blog post series, we are now dealing
    with a real EmberJS application and see how it works with ES6 modules
  seo_keywords: Ember, Ember js, EmberJS, ES6 modules, Ember modules
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

> **Update 28/06/2014**
> Ember App Kit is now deprecated in favor of [Ember-CLI](http://www.ember-cli.com/)

EmberJS does not have a built-in module system per-se and the community is now pushing toward ES6 modules as the standard for creating applications with EmberJS.

![ember-structure-sm](/assets/img/ember-structure-sm.png)

I've been looking at the Ember App Kit (EAK) for a couple of weeks now, and I'm solving big problems with it ! Both in terms of productivity and scalability. Creating AMD modules with Ember's conventions becomes a breeze to write thanks to the Resolver shipped with EAK. You first need to understand how things work, how files make sense together... but once you'll get that, you'll start to love it.

For instance your route <code class="language-markup">routes/users.js</code> will know to use the controller in <code class="language-markup">controllers/users.js</code> and the template in <code class="language-markup">templates/users.hbs</code>. You might want to manually add a model somewhere, you can use an <code class="language-javascript">import</code> statement to get the job done !

## Nuff' talk ! Gimme some code !
All right, all right, we'll implement the "Mailbox" example from the EmberJS website using ES6 modules, sounds good ? Keep reading ;) Clone the git [repo](https://github.com/thaume/emberjs-es6-modules/tree/ember-es6-1) and follow me !

Here is the structure we'll need (for the **app/** folder) :

{% highlight javascript %}
app /
|---- app.js
|---- router.js
|---- helpers /
|-------- mail-date.js
|---- models /
|-------- mailbox.js
|---- routes /
|-------- application.js
|-------- mail.js
|---- templates /
|-------- application.hbs
|-------- index.hbs
|-------- mail.hbs
|-------- mailbox.hbs
|-------- maibox /
|------------ index.hbs
{% endhighlight %}

Simple right ? Well that's enough to get the party startin' ! Keep in mind that for ES6 modules, only the paths matters : what you were used to name 'UsersRoute.js' will become 'route/users.js'.

## The router
The router is the heart of our application, here is how it looks with ES6 modules :

{% highlight javascript %}
var Router = Ember.Router.extend(); // ensure we don't share routes between all Router instances

Router.map(function() {
  this.resource('mailbox', { path: '/:mailbox_id' }, function() {
    this.resource('mail', { path: '/:message_id' });
  });
});

export default Router;
{% endhighlight %}

We declare a local variable Router (local in the module) and then export it to make it reachable as a module. We declared two routes :
<ol>
  <li>"mailbox", resolving to /:mailbox_id</li>
  <li>"mail", resolving to /:message_id</li>
</ol>

The dynamic parts of the URL will be the different filters that we will apply to display the mails (inbox, spam, sent mail).

## The routes
We need to define a behavior for the routes we just mapped to our Router, for that we'll need :

<ol>

<li>The application route callback : 
{% highlight javascript %}
  import Mailbox from 'appkit/models/mailbox';

var ApplicationRoute = Em.Route.extend({
  model: function() {
    return Mailbox.find();
  }
});

export default ApplicationRoute;
{% endhighlight %}</li>

<li>The mail route callback :
{% highlight javascript %}
var MailRoute = Em.Route.extend({
  model: function(params) {
    return this.modelFor('mailbox').messages.findBy('id', params.message_id);
  }
});

export default MailRoute;
{% endhighlight %}</li>

</ol>

We need to import the model 'mailbox' to be able to query all the messages in the application route, but we can use the <code class="language-javascript">modelFor('mailbox');</code> method in the mail route to use the model of mailbox which was imported as context in the <code class="language-markup">{{ "{{#link-to" }}}}</code> as you'll see later in the application.hbs, mailbox.hbs templates.

## The model
In the application route, we had to import the model Mailbox in order to query all the messages of our mailbox. Here is the model :

{% highlight javascript %}
var Mailbox = Em.Object.extend();

Mailbox.reopenClass({
  find: function(id) {
    if (id) {
      return FIXTURES.findBy('id', id);
    } else {
      return FIXTURES;
    }
  }
});

export default Mailbox;

var FIXTURES = [
  ...
];
{% endhighlight %}

Here we created an object model that will give the data to the route. We've been able to import this model in the application route thanks to the <code class="language-javascript">export default Mailbox</code>. I didn't wanted to display the FIXTURES but you'll find them in the repo I gave you at the beginning of the article.

## Templates
<strong>application.hbs:</strong> The first template that will be rendered is application.hbs, on the root URL <code class="language-markup">/</code> . Ember will automatically link the template to the route based on names, and application.hbs + index.hbs come as the "default frame" when you kickstart an application :

{% highlight HTML %}
<div class="url">URL: {{ "{{target.url" }}}}</div>
<aside>
    <ul>
        <li><h2>Mailboxes</h2></li>
        {{ "{{#each" }}}}
            <li>
                {{ "{{#link-to "mailbox" this currentWhen="mailbox"" }}}}
                    <span class="count">
                        {{ "{{messages.length" }}}}

                    {{ "{{name" }}}}
                {{ "{{/link-to" }}}}
            </li>
        {{ "{{/each" }}}}
    </ul>
</aside>

<section class="main">
    {{ "{{outlet" }}}}
</section>
{% endhighlight %}

<strong>index.hbs:</strong> Then Ember will request the index.hbs template and will render it in the {{ "{{outlet}}" }} of application.hbs:

{% highlight HTML %}
<div class="index">
  <h1>TomsterMail</h1>
  <span class="tomster">
</div>
{% endhighlight %}

<strong>mailbox.hbs: </strong>Once you'll click on either 'inbox', 'spam' or 'sent mail' (links that will be displayed in the {{#link-to}} area in application.hbs), the mailbox.hbs template will be called and displayed in the outlet of application.hbs :

{% highlight HTML %}
<table>
    <tr>
        <th>Date</th>
        <th>Subject</th>
        <th>From</th>
        <th>To</th>
    </tr>

    {{ "{{#each messages" }}}}
        {{ "{{#link-to "mail" this tagName='tr'" }}}}
            <td>{{ "{{mail-date date" }}}}</td>
            <td>{{ "{{view.isActive" }}}}{{ "{{subject" }}}}</td>
            <td>{{ "{{from" }}}}</td>
            <td>{{ "{{to" }}}}</td>
        {{ "{{/link-to" }}}}
        </tr>
    {{ "{{/each" }}}}
</table>

{{outlet}}
{% endhighlight %}

<strong>mailbox/index.hbs: </strong>The mailbox/index.hbs template will be rendered as the default template in the {{outlet}} of mailbox.hbs

{% highlight HTML %}
<div class="mailbox-index">
  Select an email
</div>
{% endhighlight %}

<strong>mail.hbs: </strong>And then, once you'll click on one of those mails you've received, the mail.hbs template will be rendered and populated with the mail's data

{% highlight HTML %}
<div class="mail">
  <dl>
      <dt>From</dt>
      <dd>{{ "{{from" }}}}</dd>
      <dt>To</dt>
      <dd>{{ "{{to" }}}}</dd>
      <dt>Date</dt>
      <dd>{{ "{{mail-date date" }}}}</dd>
  </dl>
  <h4>{{ "{{subject" }}}}</h4>
  <p>{{ "{{body" }}}}</p>
</div>
{% endhighlight %}

The templates are processed by the grunt task <code class="language-markup">grunt-ember-template</code>, you should not have to worry about that, just use the classic Ember's conventions names for your templates (just remember to switch to a pathname style convention).

We now have everything we need to kickstart our app, let's write the <code class="language-markup">app.js</code> and unleash Ember !

## App.js
Our app.js file is the entry point of our application, where we create the application and launch the Resolver (all the LOG_* are for debug purpose):

{% highlight javascript %}
import Resolver from 'resolver';

var App = Ember.Application.extend({
  LOG_ACTIVE_GENERATION: true,
  LOG_MODULE_RESOLVER: true,
  LOG_TRANSITIONS: true,
  LOG_TRANSITIONS_INTERNAL: true,
  LOG_VIEW_LOOKUPS: true,
  modulePrefix: 'appkit',
  Resolver: Resolver['default'],
  rootElement: "#target"
});

Ember.RSVP.configure('onerror', function(error) {
  if (error instanceof Error) {
    Ember.Logger.assert(false, error);
    Ember.Logger.error(error.stack);
  }
});

export default App;
{% endhighlight %}

Once all your modules are exported, they will go throught the Resolver and will be given the place they should have : the router, the routes, the templates, the helpers... It is all (almost) magical and forces you to be idiomatic in many ways (You will not be able to call a global controller in a template, you'll have to make things clean).

<strong>One last thing: </strong>for now, you'll need to call explicitely the app.js default method in your index.html :

{% highlight javascript %}
<script>window.App = require('appkit/app')["default"].create();</script>
{% endhighlight %}

The last thing you need to do now is to run <code class="language-bash">grunt server</code> in your terminal (from the root of the [git you cloned](https://github.com/thaume/emberjs-es6-modules)), and let the Ember App Kit drive you throught ES6 modules transpiled to AMD inside of your browser, have fun !

I want a [DEMO LINK](http://thau.me/demo/emberjs-es6/) !

- [ES6 modules and EmberJS : a taste of the future (part 1)](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-1/)
- [ES6 modules and EmberJS : a taste of the future (part 2)](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/)
