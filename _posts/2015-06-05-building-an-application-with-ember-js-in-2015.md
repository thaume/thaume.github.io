---
layout: talks
title: Building an application with Ember.js in 2015
date: 2015-06-05 18:53:51.000000000 +01:00
type: post
talk: true
published: true
meta:
  seo_description: The revolution of building Ember application in 2015
---

<section>
  <h1>Building an application with Ember.js in 2015</h1>
  <p>
    <small>By <a href="http://thau.me" target="_blank">Tom Coquereau</a> / <a href="http://twitter.com/thaume" target="_blank">@thaume</a> / <a href="http://www.blaaast.co" target="_blank">Blaaast</a></small>
  </p>
</section>


<section>
  <h1>Sommaire</h1>
  <h2>La vision d'Ember et le tooling</h2>
  <ol style="width: 50%;">
    <li>Ember's goal</li>
    <li>Stability Without Stagnation</li>
    <li>Ember-CLI</li>
  </ol>
</section>


<section>

  <section>
    <h1>Ember's goal</h1>
  </section>

  <section>
    <h2>Focus sur les "Ambitious Web Applications"</h2>
    <ul>
      <li>Un framework MVC complet</li>
      <li>Ne résoud pas seulement le problème "V" de MVC</li>
      <li>The "Ember-way"</li>
      <li>Les développeurs sont rapidement à l'aise sur les projets (eg. Discourse)</li>
    </ul>
  </section>

</section>


<section>

  <section>
    <h1>Stability Without Stagnation</h1>
  </section>

  <section>
    <h2>1.13 features</h2>
  </section>

  <section>
    <h2>1.13: Moteur de rendu Glimmer</h2>
  </section>

  <section>
    <h2>1.13: One way data flow (actions up, data down)</h2>
  </section>

  <section>
    <h2>1.13: Routable component</h2>
  </section>

  <section>
    <h2>1.13: Les nouveaux "lifecycle hooks" des composants</h2>
  </section>

  <section>
    <h2>1.13: Liquid-fire 1.0-beta1</h2>
  </section>

  <section>
    <h2>1.13: Fast Boot™</h2>
  </section>

  <section>
    <h2>2.0 beta1</h2>

    <p>12 juin</p>
  </section>

  <section>
    <h2>2.0 features</h2>

    <p>Undefined is not a function</p>
  </section>

  <section>
    <h2>2.0 features</h2>

    <p>NaN</p>
  </section>

  <section>
    <h2>2.0 features</h2>

    <p>-1</p>
  </section>

  <section>
    <h2>WAT ?</h2>
  </section>

  <section>
    <h2>Pas de features dans la 2.0 ?</h2>
  </section>

  <section>
    <h2>2.0 === 1.13</h2>
    <p>Stability without stagnation</p>
  </section>

  <section>
    <img src="/assets/img/ember-upgrades.jpg">
  </section>

  <section>
    <h2>Deprecations !</h2>
    <ul>
      <li>Ajout de deprecations</li>
      <li>Suppression des features sur le major bump</li>
      <li>Donc: 2.0 === 1.13 (- deprecated features)</li>
    </ul>
  </section>

  <section>
    <h2>Comment se préparer à la 2.0 ?</h2>
  </section>

  <section>
    <h2>Préparation à la 2.0:</h2>
    <p>Upgrade sur Ember 1.13 (et fix du code deprecated)</p>
  </section>

  <section>
    <h2>Example: Deprecation of Controllers</h2>
    <h3>Single Responsibility Principle (SRP)</h3>
    <ul>
      <li>Persistance des "states" de l'app</li>
      <li>Décoration des données pour les templates</li>
      <li>=> "Quand devrais-je utiliser un controller vs un component ?"</li>
    </ul>
  </section>

  <section>
    <h2>The road to "Routable Components"</h2>
    <ul>
      <li>ObjectController deprecated dans la 1.11</li>
      <li>Controllers (Ember.Controller et Ember.ArrayController) deprecated dans la 1.13</li>
      <li>Chemin de migration: Controller => Service + Component</li>
    </ul>
  </section>

  <section>
    <h2>Services</h2>
    <p>Singletons qui peuvent contenir des "states" de l'app</p>
  </section>

  <section>
    <h2>Components</h2>
    <p>Décoration de la donnée / gestion des actions et des événements pour les templates / routable</p>
  </section>

  <section>
    <h2>Les Controllers seront supprimés dans la 2.0</h2>
    <p><a href="https://github.com/emberjs/rfcs/pull/38">https://github.com/emberjs/rfcs/pull/38</a></p>
  </section>

  <section>
    <h2>Computed properties</h2>
<pre><code data-trim contenteditable>
export default Ember.Component.extend({
  fullName: function() {
    return '' + this.attrs.firstName + this.attrs.lastName;
  }.property()
});
</code></pre>
  </section>

</section>


<section>

  <section>
    <h1>Ember-CLI</h1>
  </section>

  <section>
    <h2>The CLI revolution</h2>

    <ul>
      <li>Introduced mid-2014</li>
      <li>Bootstrap a project in seconds</li>
      <li>Generate tests alongside routes, components, etc.</li>
      <li>Addons ecosystem (1000+ addons)</li>
    </ul>
  </section>

  <section>
    <h2 class="tolowercase">$ ember new blog</h2>
    <p>Generates a full Ember app</p>
  </section>

  <section>
    <img src="/assets/img/install-app.png">
  </section>

  <section>
    <h2 class="tolowercase">$ ember g route posts</h2>
    <p>Generates the post's route and template</p>
  </section>

  <section>
    <h2 class="tolowercase">$ ember g model post title:string body:string</h2>
    <p>Generates a model 'post' with a title and a body of type string</p>
  </section>

  <section>
    <h2 class="tolowercase">$ ember generate serializer application</h2>
    <p>Generates a serializer</p>
  </section>

  <section>
    <h2 class="tolowercase">$ ember g http-mock post</h2>
    <p>Generates a 'post' http-mock in the node.js server</p>
  </section>

  <section>
    <h2>Ember addon</h2>
    <p><a href="http://www.emberaddons.com/" target="_blank">http://www.emberaddons.com/</a></p>
    <p>ember install ${addonName}</p>
  </section>

</section>

<section>
  <h1>Questions ?</h1>
  <h3>
    <small>By <a href="http://thau.me" target="_blank">Tom Coquereau</a> / <a href="http://twitter.com/thaume" target="_blank">@thaume</a> / <a href="http://www.blaaast.co" target="_blank">Blaaast</a></small>
  </h3>
</section>
