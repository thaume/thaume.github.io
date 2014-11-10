---
layout: post
title: Ember Data - Mastering async relationships part 2
date: 2014-11-07
type: post
published: true
tweetId:
meta:
  seo_title: Ember Data mastering async relationships - part 2
  seo_description: A walkthrough Ember Data async relationships, creating and displaying a new project with the help of Ember-CLI.
  seo_keywords: emberjs, ember-data, ember-cli, async, relationships, create, new, write, writing, adding, pushObjects
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

This post is the second part of "Ember Data: Mastering async relationship", you will find the first part [here](/2014/09/ember-data-mastering-async-relationships/). It will cover the behaviour of async relationships while communicating with a CRUD back-end.

## Requirements

To follow this blog post, you need a basic understanding of how Ember.js and Ember Data work, if you want to discover those framework with a simpler project, you can read [this post](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/).

You'll also need the ember-cli toolbelt installed, you can find all the instructions [here](http://www.ember-cli.com/).

## Get the bootstrapped app

The bootstrapped app is available [here](https://github.com/thaume/ember-data-async-rel). You'll need it to go through the steps of this tutorial (mostly for the server-side code).

Once you've downloaded it, switch to the 'async-crud' branch and proceed with the classic `npm install && bower install`. Then fire `ember server` to start the whole thing.

## Context

For this blog post, we're going to create the "write" part of our project management application:

- **The project creation** screen: Create a new project and attach it to the `currentUser`. We will also add a `participant` array to this newly created project. This will be the `projects.new` route.

## Models

We already created the 2 required models in the [previous blog post](/2014/09/ember-data-mastering-async-relationships/):

- The `User` model
- The `Project` model

## The project creation screen

For our users to be able to create new projects, we'll need to offer them a `<form>` where they can actually enter a `title`, a `description` and add some participants (which are users).

The classic strategy is to create a `projects.new` route with its corresponding template (`templates/projects/new.hbs`) and controller (`controllers/projects/new.js`).

### The 'projects.new' route

First, we'll add this new route to our `router`:

{% highlight javascript %}
import Ember from 'ember';

var Router = Ember.Router.extend({
  location: EdAsyncRelENV.locationType
});

Router.map(function() {
  this.resource('projects', function () {
    this.route('new'); // this is the line I'm talking 'bout

    this.resource('project', { path: ':id' }, function () {
      this.route('participants');
    });
  });
});

export default Router;
{% endhighlight %}

Then we'll need to create the route file (in `routes/projects/new.js`), it will look like this:

{% highlight javascript %}
import Ember from 'ember';

export default Ember.Route.extend({

  model: function () {
    return Ember.RSVP.hash({
      newProject: this.store.createRecord('project', {
        title: '',
        description: ''
      }),
      users: this.store.find('user')
    });
  }

});
{% endhighlight %}

What's happening here? We need to push two objects (users and newProject) to the controller so that he can both manage the newly created project record and also get a list of users to show 'selectable' participants in our `<form>` (this is a demo, you might not want to show all your database users in a real-world app).

### The 'projects.new' template

Now that we have all the data needed, let's build our `<form>`! Here is the code from `templates/posts/new.hbs`:

{% highlight html %}
<form role="form" {{ "{{action 'create' on='submit'" }}}}>
  <legend>Create a new project</legend>

  <p>
    <label for="title">Project name</label>
    {{ "{{input type='text' name='Project name' value=newProject.title" }}}}
  </p>

  <p>
    <label for="description">Project description</label>
    {{ "{{input type='text' name='Project description' value=newProject.description" }}}}
  </p>

  <p>
    <label>Pick the participants</label>
    <ul>
    {{ "{{#each model.users" }}}}
      <li>{{ "{{input type='checkbox' checked=this.checked" }}}} {{ "{{name" }}}}</li>
    {{ "{{/each" }}}}
    </ul>
  </p>

  <button type="submit" class="btn btn-default">Create project</button>

</form>
{% endhighlight %}

We now need to bind our newProject records properties in its corresponding input's value attribute (title, description).

Then, in order to be able to select the users we want as `participants`, we can use 2-way data-binding and push a new property `check` on each user. This is a bit lame (to do it right you might want to use components instead) but it is much simpler for this demo.

Now that our template is ready to receive data, let's create an `action` to notify our controller that the user is submitting the form. This is what is happening in the `<form>` tag. The `{{ "{{action 'create' on='submit'" }}}}` will call the `create` action method on our controller (`controllers/projects/new.js`) when the form triggers the `submit` event. Let's see how that looks on our controller.

### The 'projects.new' controller

This is where we'll merge all the data needed to complete our new project record and sync it with our REST API.

First, we need to `alias` our model properties so that `users` and `newProject` are merged in the controller's scope. This operation is easily done with `Ember.computed.alias`:

{% highlight javascript %}
...
users: Ember.computed.alias('model.users'),
newProject: Ember.computed.alias('model.newProject'),
...
{% endhighlight %}

Then we need to handle the `create` action triggered from our `<form>`. We need to create an `action` hash where we'll add a `create` method like so:

{% highlight javascript %}
actions: {
  create: function () {
    ...
  }
}
{% endhighlight %}

In this create method we need to do 3 things: get our selected users, save the newProject record and transition to the newProject's route.

#### 1 - Getting the projects participants
We need to filter our `users` array to get the selected users (once again, this is for the purpose of this demo, in real life you might want to use a component to handle checkboxes actions). The filter method looks like this:

{% highlight javascript %}
actions: {
  create: function () {
    // Creation of the array of selected users
    var selectedUsers = this.get('users').filter(function (user) {
      return !!user.get('checked');
    });
  }
}
{% endhighlight %}

At that point, `selectedUsers` is an array of users that have a property `checked` set to `true`. Those are the users we'll want to inject into our `participants` relationship.

#### 2 - Complete and save the record

In order to complete our record, we need to inject our `selectedUsers` into the `newProjects.participants` relationship. This relationship being asynchronous, we need to first "open" it. An asynchronous hasMany relationship is a `PromiseArray`, wich means that `newProject.participants` is a thenable.

If you try to push objects into an async relationship without first "opening" it, it will warn you that the property is read-only.

Opening an async relationship looks like this:

{% highlight javascript %}
// Opening the PromiseArray 'participant' from the newly created project
return this.get('newProject.participants').then(function (participants) {
  // We now have access to the newProject.participants property
  // and it is not read-only here
  ...
});
{% endhighlight %}

Then, we need to push our `selectedUsers` array into `newProject.participants`. We'll do it like that:

{% highlight javascript %}
return this.get('newProject.participants').then(function (participants) {
  // Participants is an array where we can push objects
  participants.pushObjects(selectedUsers);

})
{% endhighlight %}

Finally, we can save our record which now has the `selectedUsers` array set to its `participants` relationship (btw don't trust me, check in your ember inspector).

You'll also see that the payload sent from the `.save()` method will feature an array of the selected users' ids.

{% highlight javascript %}
// Opening the PromiseArray 'participant' from the newly created project
return this.get('newProject.participants').then(function (participants) {
  participants.pushObjects(selectedUsers);
  return this.get('newProject').save();

}.bind(this))
{% endhighlight %}

#### 3 - Transition to the newProject's route

Once our model is created on the server (don't forget to send a `201` http code) we can safely transition to our newly created project route.

{% highlight javascript %}
actions: {
  create: function () {
    // Creation of the array of selected users
    var selectedUsers = this.get('users').filter(function (user) {
      return !!user.get('checked');
    });

    // Opening the PromiseArray 'participant' from the newly created project
    return this.get('newProject.participants').then(function (participants) {
      participants.pushObjects(selectedUsers);
      return this.get('newProject').save();

    }.bind(this)).then(function (project) {
      this.transitionToRoute('project.index', project.get('id'));

    }.bind(this));
  }
}
{% endhighlight %}

Btw I don't think you need to wait for the server's response, but that might be the content of another UX-centred blog post.

## Wrapping up

You can now enjoy your newly created project and check if the participants you selected are present! Once you reload your page, the project will of course disapear.

Respond to this tweet to talk about this post.
