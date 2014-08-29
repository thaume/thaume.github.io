---
layout: post
title: Entering the world of SASSC and bourbon for a real project
date: 2013-11-05 19:36:18.000000000 +01:00
type: post
published: true
meta:
  seo_title: Intro to Libsass and bourbon, the C powered SASS wrapper
  seo_description: Sass can be slow sometimes, in this article we are looking
    at an alternative to the classic sass/compass
  seo_keywords: Sass performance, grunt, grunt-sass, node-sass, libsass, bourbon,
    workflow
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---
We are all starting to 'build' our front-end resources. Some of us pre-compile Handlebars templates, some minify images and I even heard that some deal with [cache-busting](https://github.com/jgallen23/grunt-hash). We run all of those tasks thanks to this amazing tool called 'Grunt' that keeps on revolutionizing the front-end way of building websites.

Of course we both want to run these build tasks and use livereload to design in the browser. For a nice developer-experience, it needs to run **fast**.


Most of these tasks are okay in terms of speed, but when your Sass/Compass project reaches 4000 lines, you start to have performance issues (at the compile step and not in the css generated, this is another story).

I reached those 4000 lines of code on a project I was working on earlier this year. It took around 5 seconds to compile the scss files with compass. One of the guy working with me on the project ([@damln](https://twitter.com/damln)) told me about this new Sass built with C, it sounded really sassy.

At that point, I met [Libsass](https://github.com/hcatlin/libsass) (it's a good guy)

[![Libsass image](/assets/img/libsass-logo.png)](http://libsass.org/)

## Enter libsass and Bourbon

I then started a complete re-factoring of the code using Bourbon as my 'mixins-provider' and building the scss with Libsass wrapped in a grunt-task (<a title="grunt-sass on github" href="https://github.com/sindresorhus/grunt-sass">grunt-sass</a>, and not grunt-contrib-sass which is based on the ruby version). The good thing about Bourbon is that their mixins are w3c compliant (something not absolutely true in compass) and they drop support for vendor-prefix at a good pace depending on each feature, your css is cleaner and up to date !

Dropping Compass for Bourbon was already a big win, the scss compile time went from 5 sec to around 2 sec.

### So how does that look like ?

Compilation with **sass** without compass (the ruby version) :

![ruby sass compilation](/assets/img/sass_compile.png)

Compilation with **sassc** (the C version) :

![ruby sass compilation](/assets/img/sassc_compile.png)

We came from 2290 ms to 288 ms ! This is a **huge** win for people who love to design in the browser.

### What about the cons ?
<blockquote>
  <strong>Update 01/03/2014</strong>
  <br>Libsass now supports sourcemaps
</blockquote>

If you look around in the github repos of libsass, grunt-sass or node-sass, you will find out that some features are missing or partially implemented. For instance, you won't be able to take advantage of the nested @extend rules [yet](https://github.com/hcatlin/libsass/issues/159) (the single class @extend works thought), sourcemaps are in the [pipe](https://github.com/hcatlin/libsass/issues/122%23issuecomment-21885955) but not there yet either.

Considering that, I never really had a blocking problem with libsass and it is 100% compatible with bourbon and [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass). You sometimes might have to use some tricks to compile some old properties like this one :

{% highlight css %}
.identifiant {
  filter: alpha(opacity=50);
}

/* To make the compilation pass */
.identifiant {
  filter: unquote("alpha(opacity=50);");
}
{% endhighlight %}

You will get all the assistance you need on the issue thread of the github repo dealing with libsass.

## Hold your breath and...

Try libsass ! Installing the binaries is pretty easy (explanations [here](https://github.com/hcatlin/sassc)) and you can just try it with the grunt task :

{% highlight bash %}
npm install grunt-sass
{% endhighlight %}

and then add the task to your project gruntfile, like in this example : [grunt-sass example](https://github.com/thaume/sassc-test)

The roadmap of libsass is moving toward a 1:1 equivalence with the ruby version, and it is already looking pretty good, using libsass is something really refreshing !

<blockquote>
  Thanks <a title="Damian twitter" href="https://twitter.com/damln">@damln</a> for <a title="Dam's libsass article" href="http://www.damln.com/log/sassc-and-bourbon-it-works/" target="_blank">sharing your knowledge on libsass</a> ;)
</blockquote>
