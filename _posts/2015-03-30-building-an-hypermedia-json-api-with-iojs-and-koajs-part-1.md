---
layout: post
title: Building an Hypermedia JSON API with io.js and koa.js (part 1)
date: 2015-03-30
type: post
published: true
tweetId: 598414198478090240
meta:
  seo_title: Building an Hypermedia JSON API with io.js and koa.js (part 1)
  seo_description: How to build a full API with io.js and koa.js
  seo_keywords: node.js, io.js, iojs, koa, koa.js, api, hypermedia, rest, json, oauth2, acl
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

As a guy who've worked for some time with node 0.11 and koa, it is now really refreshing to have generators and a couple other ES2015/ES6 features working right out of the box. I will focus this blog posts series around [io.js](http://iojs.org) and [koa.js](http://koajs.com/) and global API design paterns.

We will be looking at quite a lot of what a modern API is supposed to be able to do (from OAuth2 to ACL and ORM). Although I will try my best to describe them, do not hesitate to contact me on [twitter](http://twitter.com/thaume) if you need more explanation.

This first post will be an exposition of the architectural principles I do find powerful while building an API and that I will use in the rest of the posts series.

## Context, architecture and some thoughts on hypermedia JSON APIs

*"But dude, hypermedia is just another buzzword"*

Well it is, when really it shouldn't be: it is the very basics of HTTP communication. Nothing as fancy as REST APIs, hypermedia can be defined pretty easily:

> "Hypermedia is the entity that defines how you can **discover media** via **links**."

Hypertext links are just hypermedia entities used to navigate from documents to documents. Boom.

### The JSON API

The JSON API on the other hand is a standard that defines how you should handle the hypermedia "states" and "data". I've been working with it for a couple of months now and it works really great. The basic idea is that you should be able to cache as much fetched records as possible on the client, to do so you need to know two things: what is the `type` of the model and what is his unique `id`.

To do so, here is the proposed data structure:

{% highlight javascript %}
{
  "data": [{
    "id": "10",
    "type": "posts",
    "title": "Rails is omakaze",
    "links": {
      "author": {
        "related": "http://mydomain.com/people?postId=10",
        "linkage": {
          "id": "9"
          "type": "people"
        }
      },
      "comments": {
        "related": "http://mydomain.com/comments?postId=10"
      }
    }
  }, {
    "id": "14",
    "type": "posts",
    "title": "Rails is omakaze"
    ...
  }],
  "links": {
    "self": "http://mydomain.com/posts",
    "next": "http://mydomain.com/posts?page[offset]=2",
    "last": "http://mydomain.com/posts?page[offset]=10"
  },
  "included": [{
    "id": 9,
    "type": "people",
    "firstname": "Joe",
    "lastname": "Doe"
  }]
}
{% endhighlight %}

#### Getting the related author
If your client-side application is compatible with this API standard, you will not even have to fetch the author of the post since it is bundled in the data returned by the API. But you still have the URL if you want to reload (re-fetch, refresh) the author of the post. Next time you need a record with `{ type: 'people' }` and `{ id: 9 }` it will be loaded from the client-side application cache, no need to ask the API.

#### Getting the related comments
On the other hand, the comments are not loaded so you need to use the `related` link to fetch the comments of this post. The API that sent this JSON could have chosen to include the `comments` relationship of the `posts` but decided that it was better to load it asynchronously once the post is displayed (or maybe the user needs to click on some button to actually load the comments).

Those are the basics of the JSON API. This little JSON provides two things: an easy way to cache records and an easy way to 'discover' related records. Awesome !

If you want to learn more you should visit [the JSON API website](http://jsonapi.org) (MIME type, complete data structure...).

### Architecturing our API

The first thing you need to decide when it comes to architecturing APIs is the way you'll want to handle URLs. Rails taught me that the best way to do it is with this structure: `posts/1/comments`. Although I've used this kind of URL a lot, I now prefer to use URLs of the form `comments?postId=1`. I like how it makes the 'post hasMany comments' relationship not essential to the life of the 'comments' resource.

For the rest of the posts series, I'll use the `comments?postId=1` URL handling form; if you do not like it you can mentaly switch to the `posts/1/comments` form ;)

#### Resources and tests

Keeping tests close from the resources actions handling is important; that way if you start to have a lot of resources you won't lose yourself in a sea of lost files... They should both live in an `/api` folder like so:

{% highlight javascript %}
-- MyApiProject /
-- -- api /
-- -- -- posts /
-- -- -- -- config.js
-- -- -- -- index.js
-- -- -- -- test.js
-- -- -- comments /
-- -- -- -- config.js
-- -- -- -- index.js
-- -- -- -- test.js
{% endhighlight %}

#### Models

Models are the solid center of your API; this is where records creation, edition, serialization and validation will happen. We'll see how a simple ORM can help you get a clean and concise database communication layer.

#### Database migrations and seeds

While in a past job I had, we had no way to replicate the database localy and it was a huge pain to push 'working' data inside it. I really suffered from that and once I found myself in the situation of building an API from scratch, my first concern was to use a solid database schema management and seed.

We'll see how the query builder [Knex.js](http://knexjs.org/) can help us manage database schemas and create seeds that can be really useful for testing and poping new environments.

#### Services

Sometimes the api endpoints (or web controllers) will need to perform some actions that are a bit out of scope for the web controller (imagine when you save your post you then want to asynchronously create an 'analytics' record to push posts visits). Doing so would be a bit heavy in the web controller 'posts', so you might want to push this kind of logic into what I like to call a 'service'.

We'll see how they can make your api more solid and easier to test.

#### Security

Designing a security-first API is important, even when starting your database schema you'll need to think about how records can be created, edited, read and deleted. The last part of this tutorial will be about ACLs and roles checking that do not kill the API caching system.

All the "real" and "interesting" content will be coming soon ! Stay tuned !
