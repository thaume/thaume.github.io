---
layout: page
title: Projects
date: 2014-08-09 14:06:32.000000000 +02:00
type: page
published: true
meta:
  seo_title: ''
  seo_description: ''
  seo_keywords: ''
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---
My main project these days is [Paperchain](http://www.paperchain.co), I built it with a friend who is a designer. We created this project as a POC (proof of concept) to test our ability to create a full web project. We spent almost one year thinking about this project and created the whole thing within 3 months (from concept/wireframes to working Beta).

[![Paperchain logo](assets/ref-04-paperchain-300x159.png)](http://www.paperchain.co)

## Technologies

We use NodeJS for our server-side code, with a PostgreSQL database to store our data. This stack works really well on Heroku, although we do not have 9000 requests per minute, we're not at all 'at scale'. The development process is really enjoyable.

The front-end is based on Backbone for now, but we are looking at Ember to push the website further and make it an hybrid application (able to work on iOS/Android/windows phone and web browser).

We also have a redis datastore running to store our sessions and a few cached object from the database; Amazon S3 is our cheap scalable image store and of course the whole application is versioned through Git on Github.

The deployment process is also really enjoyable, front-end and back-end tests run on our preproduction environment and the application is promoted to production once all tests are green. This give us a really smooth push-to-production process, add A/B testing to the sauce and you're never scared anymore to launch a feature at an early stage in production!
