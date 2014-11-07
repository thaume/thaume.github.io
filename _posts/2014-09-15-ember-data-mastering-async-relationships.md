---
layout: post
title: Ember Data - Mastering async relationships part 1
date: 2014-09-15
type: post
published: true
meta:
  seo_title: Ember Data mastering async relationships
  seo_description: A walkthrough Ember Data async relationships, fetching, saving and displaying everything on the templates with the help of Ember-CLI.
  seo_keywords: emberjs, ember-data, ember-cli, async, relationships
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

Although I have achieved quite a bit of work alongside Ember.js, Ember Data is a whole new world; a world of wonders for sure, but still another big step toward full mastering of the Ember framework.

I have to confess that Ember Data's (ED) async relationships gave me a hard time. It was worth the pain though, as soon as you understand how those relationships work it will get a lot easier to achieve almost anything (model-related) with a better granularity.

This post will cover the bootstrapping of the application and the first interaction (read) with an async relationship. The next post will be about modifying (write) an async relationship and syncing it with the server.

## Requirements

To follow this blog post, you need a basic understanding of how Ember.js and Ember Data work, if you want to discover those framework with a simpler project, you can read [this post](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/).
You'll also need the ember-cli toolbelt installed, you can find all the instructions [here](http://www.ember-cli.com/).

## Get the bootstrapped app

The bootstrapped app is available [here](https://github.com/thaume/ember-data-async-rel). You'll need it to go through the steps of this tutorial (mostly for the server-side code). Once you've downloaded it, process with the classic 'npm install && bower install'. Then 'ember server' to start the whole thing.

## Context

For this blog post, we're going to create a simple project management app that will contain the following screens:

- **The dashboard**: Display all projects attached to the 'currentUser' ('index' route)
- **The project description**: Show one project and its description ('projects/:id' route)
- **The project participants**: Display 'participants' of this project ('projects/:id/participants' route)

## Models

We'll need 2 models to create this product:

- The **User** model
- The **Project** model

### User model

We'll first generate the User model as we'll need it as a relationship in our Project model. To achieve that you need to enter this command in your terminal (the 'g' stands for 'generate'):

{% highlight bash %}
$ ember g model User name:string
{% endhighlight %}

This will create a model in app/models/user.js with the default property 'name' and a test in tests/unit/models/user-test.js.

{% highlight javascript %}
import DS from 'ember-data';

export default DS.Model.extend({
  name: DS.attr('string')
});
{% endhighlight %}

### Project model

The same way we created our User model, let's generate the Project model:

{% highlight bash %}
$ ember g model Project title:string description:string
{% endhighlight %}

Now we need to add our async hasMany relationship with the user through a 'participants' property. This goes as follows:

{% highlight javascript %}
import DS from 'ember-data';

export default DS.Model.extend({
  title: DS.attr('string'),
  description: DS.attr('string'),
  participants: DS.hasMany('user', { async: true })
});
{% endhighlight %}

### Why are we going async on the 'participants' relationship?

Using an async relationship inside a model lets the Ember app only fetch the data it needs depending on which route it is on. As we'll see in this post, the 'participants' array is not needed until we actually reach the 'project.participants' route.

The nice thing is that we can still define all the relationships belonging to a model and then fetch each relationship once it is needed.

### What exactly is 'participants' on the Project model?

The async relationship behave the same as a computed property. As long as you do not use it, it remains 'not calculated' and as soon as you use it (on a template or inside a function) it gets 'calculated' and you gain access to its value.

## The dashboard screen

### The 'index' route

When we'll want to display our projects on the dashboard screen, we'll need to fetch them through the model hook of our ‘index' route. Let's go ahead and generate this route:

{% highlight bash %}
$ ember g route index
{% endhighlight %}

This command will create the 'index' route, its template and its test file. We now need to fetch the projects from the model hook:

{% highlight javascript %}
import Ember from 'ember';

export default Ember.Route.extend({

  model: function () {
    return this.store.find('project');
  }

});
{% endhighlight %}

This will issue an ajax request to 'GET /api/projects' serialize the incoming payload and send the model right to the controller who will make it available to the template.

### Incoming payload

In Ember Data, you need to sideload your data in order for your model to serialize it the right way. This is what comes from the server when you have a non-async hasMany relationship:

{% highlight javascript %}
{
  projects: [{
    id: 1,
    participants: [1, 2]
  }],
  users: [{
    id: 1,
    name: 'Bruce'
  }, {
    id: 2,
    name: 'Tom'
  }]
}
{% endhighlight %}

There's no difference in structure for when you fetch them through async relations, the difference is that both arrays do not have to get there at the same time. For our example, loading the 'index' route will only require:

{% highlight javascript %}
{
  projects: [{
    id: 1,
    participants: [1, 2]
  }]
}
{% endhighlight %}

and then once we need to display the 'participants' of the project:

{% highlight javascript %}
{
  users: [{
    id: 1,
    name: 'Bruce'
  }, {
    id: 2,
    name: 'Tom'
  }]
}
{% endhighlight %}

Once fetched they will be associated with their project.

### The template

Now let's display our projects on the index template, go to /templates/index.hbs and insert the following code:

{% highlight HTML %}
<ul>
  {{ "{{#each model" }}}}
  <li>{{ "{{link-to title 'project' this" }}}}</li>
  {{ "{{/each" }}}}
</ul>
{% endhighlight %}

This will show links leading to each projects' page. It is important to understand that at that point, the 'participants' **have not been fetched yet**.

## The project description screen

### The 'project' route

Let's generate our 'project' route:

{% highlight bash %}
$ ember g route project
{% endhighlight %}

Ember-cli generated all the required files and also modified the Router of our app like this:

{% highlight javascript %}
Router.map(function() {
  this.route('project');
});
{% endhighlight %}

It did not need to change when we created the 'index' route (as this route is automatic in Ember), but for the 'projects' route, a route must be mapped in the Router.

We need to bind the 'project' route to the 'project/:id' path. Let's set it like that:

{% highlight javascript %}
Router.map(function() {
  this.resource('project', { path: 'project/:id' });
});
{% endhighlight %}

The final route file with the model hook looks like this:

{% highlight javascript %}
import Ember from 'ember';

export default Ember.Route.extend({

  model: function (params) {
    return this.store.find('project', params.id);
  }

});
{% endhighlight %}

### Incoming payload

The model hook will make a call to 'GET /api/projects/1' and render the template.

### The template

Go to /templates/projects.hbs and insert the following code:

{% highlight HTML %}
{{ "{{#with model" }}}}
  <h1>{{ "{{title" }}}}</h1>
  <p>{{ "{{description" }}}}</p>
{{ "{{/with" }}}}
{% endhighlight %}

We now have the project displaying on its correct URL!

## The project participants screen

This screen will appear beneath the title and description of the project inside the {{ "{{outlet" }}}} of templates/project.hbs

### The 'project.participants' route

Let's generate our 'participants' template:

{% highlight bash %}
$ ember g template participants
{% endhighlight %}

Only this time we'll need to change a couple of things. First we need to move the participants template to templates/project/participants.hbs and then change the Router's 'project' resource so that it looks like this:

{% highlight javascript %}
Router.map(function() {
  this.resource('project', { path: 'project/:id' }, function () {
    this.route('participants');
  });
});
{% endhighlight %}

That way, participants is nested inside the 'project' route and we can display it in the 'project' outlet.

### The template

The template is quite simple, we are just calling the 'participants' property from the parent's route's model.

{% highlight HTML %}
<ul>
{{ "{{#each model.participants" }}}}
  <li>{{ "{{name" }}}}</li>
{{ "{{/each" }}}}
</ul>
{% endhighlight %}

Once the template reaches the property 'model.participants' it will ask the store to 'GET /api/users?ids[]=2&ids[]=3&ids[]=4' (IDs sent by the server inside the 'Project' model previously).

And this is the crucial point: the async relationship 'participants' is triggered, the ajax call made and the actual participants fetched and returned all the way to the template.

### Incoming payload

As we saw before, here is what is returned by the server:

{% highlight javascript %}
{
  users: [{
    id: 1,
    name: 'Bruce'
  }, {
    id: 2,
    name: 'Tom'
  }]
}
{% endhighlight %}

At that point we correctly display the participants of the project. If you go back to the description screen (by clicking 'show description' on the project page) you'll get back to the project index page, if you click again on 'show participants', participants will be shown but no call ajax will be made. They are now 'cached'.

Now that you're done with reading async relationships, why not learning how to write+sync them with the server "Ember Data - Mastering async relationships part 2 (coming soon)".
