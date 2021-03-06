---
layout: talks
title: Talk / Animating ember.js
date: 2014-10-21 16:16:51.000000000 +01:00
type: post
talk: true
published: true
meta:
  seo_description: A closer look at the state of animation in the Ember.js community
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
    <li>Where we are coming from</li>
    <li>Where we're heading to</li>
    <li>Demo</li>
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
    <p>Route transition (eg: <code>/posts/edit</code> -> <code>/posts/1/show</code>)</p>
    <p>Model transition (eg: <code>/posts/1/show</code> -> <code>/posts/2/show</code>)</p>
  </section>
</section>

<section>
  <section>
    <h1>Where we are coming from</h1>
  </section>

  <section>
    <h2>jQuery animations</h2>
    <ul>
      <li>jQuery animations are 'hard-coded' for certain DOM elements</li>
      <li>Hard to reuse</li>
      <li>Hard to refactor</li>
    </ul>
  </section>

</section>

<section>
  <h1>Where we're heading to</h1>
</section>

<section>
  <section>
    <h1>Ember-animated-outlet</h1>
  </section>

  <section>
    <h2>The ember-animated-outlet <code>{{ "{{outlet" }}}}</code></h2>
<pre><code data-trim contenteditable>
<h1>Application template</h1>

<div class="main">
    {{ "{{animated-outlet  name='main'" }}}}
</div>
</code></pre>
  </section>

  <section>
    <h2>The ember-animated-outlet <code>link-to</code></h2>

<pre><code data-trim contenteditable>
<h2>About template</h2>

<div class="main">
    {{ "{{#link-to-animated 'posts.show' post animations='main:slideLeft'" }}}}Introduction{{ "{{/link-to-animated" }}}}
</div>
</code></pre>
  </section>

  <section>
    <h2>The ember-animated-outlet 'transitionTo' helper</h2>

<pre><code data-trim contenteditable>
App.ApplicationRoute = Ember.Route.extend({
  showPost: function(post) {
    this.transitionToAnimated('posts.show', { main: 'slideLeft' }, post);
  }
});
</code></pre>

  </section>

  <section>
    <h2>Wrapping up</h2>
    <ul>
      <li>Good starting point for animations in Ember.js</li>
      <li>CSS animations => limited control</li>
      <li>Need to manually create an empty view for each animation</li>
      <li>Impossible to define transitions between models (eg: /posts/1/show -> /posts/2/show)</li>
    </ul>
  </section>

</section>

<section>

  <section>
    <h1>Liquid-fire</h1>
  </section>

  <section>
    <h2>What is Liquid-fire?</h2>
    <blockquote>Like Ember itself, our goal is to cultivate shared abstractions so we're free to focus on bigger and better ideas. Good defaults. Convention over configuration. Composable pieces all the way down.</blockquote>
  </section>

  <section>
    <h2>Separation of concerns</h2>
    <p>The transitions maps are defined in a <code>transitions.js</code> file at the root of <code>/app</code></p>

<pre><code>this.transition(
  this.fromRoute('projects.new'),
  this.use('toLeft')
);</code></pre>
  </section>

  <section>
    <h2>The liquid-fire <code>{{ "{{outlet" }}}}</code></h2>
<pre><code data-trim contenteditable>
<h1>Application template</h1>

<div class="main">
    {{ "{{liquid-outlet" }}}}
</div>
</code></pre>
  </section>

  <section>
    <h2>Animating between models</h2>
<pre><code data-trim contenteditable>
{{ "{{#liquid-with model" }}}}

  <h1>{{ "{{title" }}}}</h1>

  <hr>

  <p>{{ "{{content" }}}}</p>

{{ "{{/liquid-with" }}}}
</code></pre>
  </section>

  <section>
    <h2>Animating a value update</h2>
<pre><code data-trim contenteditable>
<div id="liquid-bind-demo">
  {{ "{{liquid-bind hours" }}}}:{{ "{{liquid-bind minutes" }}}}:{{ "{{liquid-bind seconds" }}}}
</div>
</code></pre>

<pre><code data-trim contenteditable>
this.transition(
  this.childOf('#liquid-bind-demo'),
  this.use('toUp')
);
</code></pre>
  </section>

  <section>
    <h2>Animating an 'if' condition</h2>
<pre><code data-trim contenteditable>
<div id="liquid-bind-demo">
  <div id="liquid-box-demo">
  <label>I'm renting a </label>
  {{ "{{view 'select' value=vehicle content=vehicles" }}}}

  {{ "{{#liquid-if isBike class='vehicles'" }}}}
    <label>
      Would you like a helmet? {{ "{{input type='checkbox' value='helmet'" }}}}
    </label>
  {{ "{{else" }}}}
    <label>License Number</label>
    {{ "{{input type='text' value=license" }}}}
    <label>License State</label>
    {{ "{{view 'select' value=state content=states" }}}}
  {{ "{{/liquid-if" }}}}

  <button type="button" class="btn btn-success">Yay, let's go!</button>
</div>
</div>
</code></pre>

<pre><code data-trim contenteditable>
this.transition(
  this.fromNonEmptyModel(),
  this.hasClass('vehicles'),
  this.toModel(true),
  this.use('crossFade', {duration: 1000}),
  this.reverse('toLeft', {duration: 1000})
);
</code></pre>
  </section>

  <section>
    <h2>Powerful custom transitions</h2>
    <p>The custom transitions are defined in the ./app/transitions directory</p>

<pre><code>import {{ "{ isAnimating, finish, timeSpent, animate, stop " }}} from "vendor/liquid-fire";

export default function fade(oldView, insertNewView, opts) {
  ...
  return AnimationPromise;
}
</code></pre>
  </section>

  <section>
    <h2>Wrapping up</h2>
    <ul>
      <li>The results of lots of talks around animations in Ember.js</li>
      <li>JS animations => full control + speed</li>
      <li>Easy and powerful transitions between models</li>
      <li>Animation logic is kept away from the presentation layer</li>
    </ul>
  </section>

</section>

<section>
  <h1>DEMO</h1>
</section>

<section>
  <h1>Conclusion</h1>
  <p>Clean separation of concerns between the presentation and animations</p>
  <p>Powerful default and custom animations map</p>
</section>

<section>
  <h1>Questions ?</h1>
</section>

