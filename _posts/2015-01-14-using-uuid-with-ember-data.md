---
layout: post
title: Using UUID with Ember Data
date: 2015-01-14
type: post
published: true
tweetId:
meta:
  seo_title: Using UUID with Ember Data
  seo_description: Why using UUID with Ember Data can lead to powerful UX improvements. A write up about the ember-cli-uuid addon and its application in a real life application.
  seo_keywords: emberjs, ember-data, ember-cli, async, uuid, id, uuids, ids, example, ux
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

In my lastest post 'Ember Data - Mastering async relationships part 2' we created a project and saved it on the server this way:

{% highlight javascript %}
actions: {
  create: function () {
    ...
    return this.get('newProject.participants').then(function (participants) {
      participants.pushObjects(selectedUsers);
      return this.get('newProject').save();

    }.bind(this)).then(function (project) {
      this.transitionToRoute('project.show', project.get('id'));

    }.bind(this));
  }
}
{% endhighlight %}

Here we save the record on the server, wait for it to return a `201` http code and then transition to the project route. We have to wait for the server to return because the id is created on the server side. If we were to create the id on the client side (considering the case of a default auto-incrementing id), we would break the database (if someone was to create a project at the same time, you might generate the same id).

The problem is that my UX developer wants to transition **immediately**, not in **200ms**.

## Generating ids client-side

I need to be able to create ids client-side, this way I can create records on the fly and I do not need to wait for my back-end response to edit the record or even to use it in a template.

I created an Ember addon to make it easy for you to switch from classical ids (auto-incrementing) to UUIDs, you can find it over [here](https://www.npmjs.com/package/ember-cli-uuid). You'll find the installation instructions in the README.md. The addon will simply add a `generateIdForRecord` hook in your base DS.Adapter so that every newly created record gets an UUID as his primary key.

## Fully mastering your records client-side

Now that your records are fully created client-side, you can have a better flow between your views. For instance, creating a record and transitioning to its `show` view doesn't require to wait for the server to create the record in the database. That is made possible because you have access to its 'final' id.

You can then do:

{% highlight javascript %}
actions: {
  create: function () {
    ...
    return this.get('newProject.participants').then(function (participants) {
      participants.pushObjects(selectedUsers);
      this.get('newProject').save();
      this.transitionToRoute('project.show', newProject.get('id'));

    }.bind(this));
  }
}
{% endhighlight %}

Here we do not wait for the `.save()` method to resolve, we just let it go asynchronously while we transition to the next route `('project.index'). This opens up a whole lot of possibilities for speeding up your application.

## Wrapping up

So now that we can generate our records ids client-side, we can make the application work without needing the server to provide more data (here, record ids).

To fully master this kind of async flow, you also need to be resilient regarding errors: what should happen if the server responds a 5xx or 4xx but the records 'lives' on the client?

I've been using this pattern in hybrid mobile apps, and it really helps creating a 'native' feeling regarding the responsiveness of your application.
