---
layout: post
title: 'ES6 modules and EmberJS : a taste of the future (part 1)'
date: 2013-12-04 00:19:54.000000000 +01:00
type: post
published: true
meta:
  seo_title: 'ES6 modules and EmberJS : a taste of the future'
  seo_description: 'A quick walk throught the fresh ES6 modules spec coupled with the front MVC framework EmberJS '
  seo_keywords: es6, es6 modules, emberjs, ember, ember js, emberjs modules, future of javascript
  seo_robots_index: '0'
  seo_robots_follow: '0'
author:
  login: Tom
  email: tom.coquereau@gmail.com
  display_name: Tom Coquereau
---

These two articles blog post series is a little walk alongside ES6 modules and EmberJS. We'll try some simple ES6 modules examples in the first part and then focus on how EmberJS is eventually getting real modules. (A minimal understanding of EmberJS might be required for part2).

- [ES6 modules and EmberJS : a taste of the future (part 1)](/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-1/)
- [ES6 modules and EmberJS : a taste of the future (part 2)](/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/)

## import { module } from 'es6/spec';

The ES.current as we know it doesn't feature a standardized module system, you cannot require some fragment of code inside another fragment of code and use it while being inside a namespace (and not a global). AMD and CommonJS came along and proposed two impressive yet incompatible solutions to let developers use modules (AMD is mostly used in the browser, while CommonJS is more used server-side). The new ES6 modules spec aims at solving this module problem and also let ES6 modules transpile to both AMD and CommonJS as a polyfill.

The ES6 modules spec is [done](https://speakerdeck.com/dherman/status-report-es6-modules) and will not change before the ES6 specs release, we can now use it and feel safe about the new semantic we learn.

<blockquote>
  But hey, what about Chrome31 ?
</blockquote>

Fragmentation is our job, that's for sure ! ES6 modules do not work in any browser as of today but AMD works everywhere and ES6 transpiler do transpile to AMD. That's the cool news about ES6 modules : there is already a lot of cool tool that enable us to transpile ES6 modules into AMD/requireJS and use it in the browser !

## Enter ES6 modules

To be able to import some code, you'll first need to export it so that it becomes available for the import commands. Let's start with a simple module structured that way :

{% highlight javascript %}
tasks/
    lib/
        prefix.js
    main.js
{% endhighlight %}

### export

Remember that only the file's path matters when you want to import a fragment of code somewhere, you'll understand that with the following examples. You can directly import a function that way:

{% highlight javascript %}
// tasks/lib/prefix.js
let willNotBeExported = 'xyz';
export function prefix(x) {
    return prefixConf + x;
}
export let prefixConf = 'app-';
{% endhighlight %}

Note that we exported a function and a variable, but all the variables in the file are accessible from within the file: the <code class="language-javascript" data-lang="javascript">let willNotBeExported</code> variable is still accessible inside the function.

You can also perform the same export at the end of your file, it is the same thing really :

{% highlight javascript %}
// tasks/lib/prefix.js
let willNotBeExported = 143;
function prefix(x) {
    return prefixConf + x;
}
let prefixConf = 'app-';

export { prefix, prefixConf };
{% endhighlight %}

We now have a <code class="language-javascript">let prefixConf</code> variable and a <code class="language-javascript">function prefix(){...}</code> exported.

### import

Let's import our module from lib/prefix into our main.js and use our prefix function from within tasks/main.js :

{% highlight javascript %}
// tasks/main.js
import { prefix } from 'lib/prefix';
console.log( prefix('users-model') );
// The function will prefix 'users-model' with 'app-'
// => app-users-model
{% endhighlight %}

With the module system, I can now use functions defined somewhere else without putting everything in a global scope, this really helps to create better code.

### export default

In the following EmberJs examples, we'll use the simple <code class=" language-markup">export default</code> way of exporting modules. It is almost the same as what we saw just before, the only difference being that you can only import one entity from the module. It goes like this:

{% highlight javascript %}
// tasks/lib/prefix.js
let willNotBeExported = 143;
function prefix(x) {
    return prefixConf + x;
}
let prefixConf = 'app-';

export default prefix;
{% endhighlight %}

When importing an export default, you do not need to use the braces :

{% highlight javascript %}
// tasks/main.js
import prefix from 'lib/prefix';
console.log( prefix('users-model') );
// The function will prefix 'users-model' with 'app-'
// => app-users-model
{% endhighlight %}

You can find the code on my github account [here](https://github.com/thaume/es6-modules-experiment).

That information should be enough for the next part: the Ember App-Kit ES6 module powered app !

- [ES6 modules and EmberJS : a taste of the future (part 1)](/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-1/)
- [ES6 modules and EmberJS : a taste of the future (part 2)](/2013/12/es6-modules-and-emberjs-a-taste-of-the-future-part-2/)
