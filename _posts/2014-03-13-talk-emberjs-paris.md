---
layout: talks
title: Talk / Modularizing Ember.js
date: 2014-03-13 16:16:51.000000000 +01:00
type: post
talk: true
published: true
meta:
  seo_description: How is Ember.js being modularized
---

<section>
  <h1>EmberJS modularization</h1>
  <h3>A taste of the future</h3>
  <p>
    <small>By <a href="http://thau.me">Tom Coquereau</a> / <a href="http://twitter.com/thaume">@thaume</a></small>
  </p>
</section>

<section>
  <h1>Summary</h1>
  <h2>Which module system for Ember ?</h2>
  <ol style="width: 50%;">
    <li>Prerequisites</li>
    <li>JS module systems</li>
    <li>Modularizing Ember with ES6 modules</li>
    <li>The Ember App-Kit project</li>
    <li>Questions</li>
  </ol>
</section>

<section>
  <section>
    <h1>Prerequisites</h1>
  </section>

  <section>
    <h2>ES6</h2>
    <p>EcmaScript version 6 done by the TC39 work group</p>
    <p>New module system (and a lot of other <a href="https://people.mozilla.org/~jorendorff/es6-draft.html">specs</a>)</p>
    <p>Should be done by the end of 2014</p>
  </section>

  <section>
    <h2>Ember Resolver</h2>
    <table>
      <thead>
        <tr>
          <th>URL</th>
          <th>Route name</th>
          <th>Controller</th>
          <th>Route</th>
          <th>Template</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><code style="color:#CC9393;">/mail</code></td>
          <td><code style="color:#CC9393;">mail</code></td>
          <td><code style="color:#CC9393;">App.MailController</code></td>
          <td><code style="color:#CC9393;">App.MailRoute</code></td>
          <td><code style="color:#CC9393;">mail</code></td>
        </tr><tr>
      </tr></tbody>
    </table>
  </section>
</section>

<!-- Example of nested vertical slides -->
<section>
  <section>
    <h1>JS module systems</h1>
  </section>

  <section>
    <h2>CommonJS modules</h2>
    <pre><code data-trim contenteditable>
// math.js
exports.add = function() {
// do stuff here
};

// main.js
var add = require('./math').add;
    </code></pre>
  </section>

  <section>
    <h2>AMD modules</h2>
    <pre><code data-trim contenteditable>
define('myModule', [
"jQuery",
"React"
], function(jQuery, React){

// do stuff here ...

});
    </code></pre>
  </section>
  <section>
    <h2>AngularJS modules</h2>
    <pre><code data-trim contenteditable>
angular.module('myModule.service', []);
angular.module('myModule.directive', []);
angular.module('myModule.filter', []);

angular.module('myModule', ['myModule.service', 'myModule.directive', 'myModule.filter']).
run(function(greeter, user) {
greeter.localize({
salutation: 'Bonjour'
});
user.load('World');
});
  </code></pre>
  </section>
  <section>
    <h2>Backbone Marionette modules</h2>
    <pre><code data-trim contenteditable>
MyApp = new Marionette.Application();

MyApp.module("MyModule", function(MyModule, MyApp, Backbone, Marionette, $, _){

// do stuff here ...

});
    </code></pre>
  </section>

  <section>
    <h2>Ember modules ?</h2>
    <p class="fragment">AMD seems like the best choice...</p>
    <p class="fragment">
      <a href="http://tomdale.net/2012/01/amd-is-not-the-answer/">AMD is not the answer</a> (by Tom Dale, co-creator of EmberJS)
    </p>
    <p class="fragment">
      Ember is not going to implement AMD modules
    </p>
  </section>
</section>

<section>
  <section>
    <h1>Modularizing Ember with ES6 modules</h1>
  </section>
  <section>
    <h2>November 2013 TC39 meeting : ES6 modules spec done</h2>
    <p>Ember is moving toward ES6 modules support in 3 steps</p>
  </section>
  <section>
    <h2>Ember core</h2>
    <p>Ruby modules => ES6 modules</p>
    <p>
      Being modularized : <a href="https://github.com/emberjs/ember.js/tree/master/packages_es6">Github thread</a>
    </p>
  </section>
  <section>
    <h2>ES6 transpiler</h2>
    <p>ES6 modules are not supported in any browser yet</p>
    <p>Transpiles to:</p>
    <ul>
      <li>AMD</li>
      <li>CommonJS</li>
      <li>Global</li>
    </ul>
    <p>Modules can work both in NodeJS and in the browser</p>
    <p><a href="https://github.com/square/es6-module-transpiler">Github repo</a></p>
    </p>
  </section>
  <section>
    <h2>Ember App Kit</h2>
    <p>Strong structure to start en Ember application</p>
    <p>Custom resolver</p>
    <p>Based on ES6 modules</p>
    <p><a href="https://github.com/stefanpenner/ember-app-kit">Github repo</a></p>
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

  <section>
    <h2>What is Ember App-Kit ?</h2>
    <p>Ember project scaffolding and tools</p>
    <p>Checkout the <a href="https://github.com/stefanpenner/ember-app-kit">Github repo</a> and follow the instructions</p>
  </section>

  <section>
    <h2>Project structure</h2>
    <pre><code data-trim contenteditable style="max-height: 600px;">
#-- app
#-- adapters
application.js
#-- components
pretty-color.js
#-- controllers
mail.js
#-- helpers
reverse-word.js
#-- models
mailbox.js
message.js
#-- routes
mail.js
#-- templates
#-- mailbox
index.hbs
application.hbs
index.hbs
mail.hbs
mailbox.hbs
#-- views
mail.js
app.js
index.html
router.js
    </code></pre>
  </section>

  <section>
    <h2>Naming conventions</h2>
    <p>EAK and ES6 modules will resolve dependencies based on files paths<br>(that's what NodeJS does too)</p>
    <p><code style="color:#CC9393;">App.MailRoute</code> would render <code style="color:#CC9393;">App.MailView</code>, EAK has the same abilities with modules</p>
    <pre><code data-trim contenteditable style="max-height: 600px;">
#-- app
#-- controllers
mail.js
#-- models
mailbox.js
message.js
#-- routes
mail.js // will know it has to use views/mail and controllers/mail
#-- templates
#-- mailbox
index.hbs
application.hbs
index.hbs
mail.hbs
mailbox.hbs
#-- views
mail.js
    </code></pre>
  </section>

  <section>
    <h2>Resolver</h2>
    <table>
      <thead>
        <tr>
          <th>URL</th>
          <th>Route name</th>
          <th>Controller</th>
          <th>Route</th>
          <th>Template</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><code style="color:#CC9393;">/mail</code></td>
          <td><code style="color:#CC9393;">mail</code></td>
          <td><code style="color:#CC9393;">App.MailController</code></td>
          <td><code style="color:#CC9393;">App.MailRoute</code></td>
          <td><code style="color:#CC9393;">mail</code></td>
        </tr><tr>
      </tr></tbody>
    </table>
    <br>
    <table>
      <thead>
        <tr>
          <th>URL</th>
          <th>Route name</th>
          <th>Controller</th>
          <th>Route</th>
          <th>Template</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><code style="color:#CC9393;">/mail</code></td>
          <td><code style="color:#CC9393;">mail</code></td>
          <td><code style="color:#CC9393;">controllers/mail</code></td>
          <td><code style="color:#CC9393;">routes/mail</code></td>
          <td><code style="color:#CC9393;">templates/mail</code></td>
        </tr><tr>
      </tr></tbody>
    </table>
  </section>

  <section>
    <h2>Writing modules</h2>
    <pre><code data-trim contenteditable style="max-height: 600px;">
// Old way, stuffing everything in a global
// ----------
App.MailRoute = Ember.route.extend({
model: function(params) {
return this.store.find('mailbox', params.id);
}
});

// New way, export an ES6 module from routes/mail.js
// ----------
export default Ember.route.extend({
model: function(params) {
return this.store.find('mailbox', params.id);
}
});

// can also be written
var MailRoute = Ember.route.extend({
model: function(params) {
return this.store.find('mailbox', params.id);
}
});
export default MailRoute;
    </code></pre>
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
