---
layout: post
title: Ember Data - Mastering async relationships part 2
date: 2014-09-22 16:16:51.000000000 +01:00
type: post
published: true
meta:
  seo_title: Ember Data mastering async relationships - part 2
  seo_description: A walkthrough Ember Data async relationships, fetching, saving and displaying everything on the templates with the help of Ember-CLI.
  seo_keywords: emberjs, ember-data, ember-cli, async, relationships
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

This post will cover the behaviour of async relationships while communicating with a CRUD back-ends.

## Requirements

To follow this blog post, you need a basic understanding of how Ember.js and Ember Data work, if you want to discover those framework with a simpler project, you can read [this post](http://thau.me/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/).
You'll also need the ember-cli toolbelt installed, you can find all the instructions [here](http://www.ember-cli.com/).

## Get the bootstrapped app

The bootstrapped app is available [here](https://github.com/thaume/ember-data-async-rel). You'll need it to go through the steps of this tutorial (mostly for the server-side code). Once you've downloaded it, switch to the 'async-crud' branch and proceed with the classic 'npm install && bower install'. Then 'ember server' to start the whole thing.

## Context

For this blog post, we're going to create the 'write' part of our project management application:

- **The project creation** screen: Create a new project and attach it to the 'currentUser' ('new' route)

## Models

We already created the 2 required models in the previous blog post:

- The **User** model
- The **Project** model

## The project creation screen

### The 'projects.new' route
