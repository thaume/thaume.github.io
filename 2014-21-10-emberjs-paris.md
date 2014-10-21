---
layout: talks
title: Talk Ember.js Paris
permalink: /2014-21-10-emberjs-paris/
---

<section>
  <h1>Animating Ember.JS</h1>
  <h3>with the help of <a href="https://github.com/ef4/liquid-fire">liquid-fire</a></h3>
  <p>
    <small>By <a href="http://thau.me">Tom Coquereau</a> / <a href="http://twitter.com/thaume">@thaume</a></small>
  </p>
</section>

<section>
  <h1>Summary</h1>
  <h2>Which animation library for Ember.js ?</h2>
  <ol style="width: 50%;">
    <li>Prerequisites</li>
    <li>jQuery animations</li>
    <li>Ember-animated-outlet</li>
    <li>Liquid-fire</li>
    <li>Questions</li>
  </ol>
</section>

<section>
  <section>
    <h1>Prerequisites</h1>
  </section>

  <section>
    <h2>The Ember <code>{{ "{{outlet" }}}}</code></h2>
    <p>Placeholder where the appropriate template is injected by the Router based on the current URL.</p>

<pre><code data-trim contenteditable>
<h1>Application template</h1>

<div class="main">
    {{ "{{outlet" }}}}
</div>
</code></pre>

  </section>

  <section>
    <h2>The Ember router</h2>
    <p>Definition of routes and resources</p>
    <p>Route transition (eg: <code>/posts/edit</code> -> <code>/posts/show/1</code>)</p>
    <p>Model transition (eg: <code>/posts/show/1</code> -> <code>/posts/show/2</code>)</p>
  </section>
</section>

<section>
  <section>
    <h1>jQuery animations</h1>
  </section>
</section>

<section>
  <section>
    <h1>Modularizing Ember with ES6 modules</h1>
  </section>
</section>

<section>
  <h1>Summary</h1>
  <h2>Which module system for Ember ?</h2>
  <ol style="width: 50%;">
    <li>Prerequisites</li>
    <li>JS module systems in the browser</li>
    <li>Modularizing Ember with ES6 modules</li>
    <li>The Ember App-Kit project</li>
    <li>Questions</li>
  </ol>
</section>

<section>
  <section>
    <h1>The Ember App-Kit (EAK) project</h1>
  </section>
</section>

<section>
  <h1>DEMO</h1>
</section>

<section>
  <h1>Conclusion</h1>
  <p>Future-proof module system</p>
  <p>EmberJS is a good match for large applications</p>
  <p>Ready for client/server code sharing</p>
</section>

<section>
  <h1>Questions ?</h1>
</section>
