---
layout: talks
title: Talk / Babel, to ES7 and beyond
date: 2015-05-07 18:53:51.000000000 +01:00
type: post
talk: true
published: true
meta:
  seo_description: The mutation of Ember to ES6/ES7 through Babel
---

<section>
  <h1>Ember and Babel, to ES7 and beyond</h1>
  <h3>A clear vision of the future</h3>
  <p>
    <small>By <a href="http://thau.me" target="_blank">Tom Coquereau</a> / <a href="http://twitter.com/thaume" target="_blank">@thaume</a> / <a href="http://www.blaaast.co" target="_blank">Blaaast</a></small>
  </p>
</section>


<section>
  <h1>Summary</h1>
  <h2>How is Ember moving to the new JavaScript ? <small>(and features you should be aware of)</small></h2>
  <ol style="width: 50%;">
    <li>What is Babel ?</li>
    <li>ES6/ES7 important features</li>
    <li>A practical Ember example</li>
  </ol>
</section>


<section>

  <section>
    <h1>What is Babel ?</h1>
  </section>

  <section>
    <h2>An ECMAScript transpiler</h2>

    <p>2009: TC-39 decides to create an ever-evolving version of ECMAScript</p>
    <p>Developers crave to use the new features (unavailable in most browsers)</p>
    <p>Babel transpiles the code so that any browser can run the modern code you type</p>
  </section>

  <section>
    <h2>Old and new name</h2>

    <p>6to5 => Babel</p>
    <p>ES6, ES7, ES2015, ES2016...</p>
    <p>Babel is future-proof</p>
    <p>You will always need Babel to use new language features</p>
  </section>

</section>


<section>

  <section>
    <h1>ES6/ES7 important features</h1>
  </section>

  <section>
    <h2>The arrow function</h2>

<pre><code data-trim contenteditable>
// ES3 way of binding scope
(function() {
  var _this = this;
  this.age = 24;

  someAsyncFunc().then(function() {
    console.log(this.age); // undefined
    console.log(_this.age); // prints 24
  });
}());

// ES5 way of binding scope
(function() {
  this.age = 24;
  someAsyncFunc().then(function() {
    console.log(this.age); // prints 24
  }.bind(this));
}());

// ES6 way of binding scope
(function() {
  this.age = 24;
  someAsyncFunc().then(() => {
    console.log(this.age); // prints 24
  });
}());

// Fancy ES6 map
const ids = rawData.map(data => data.id);
</code></pre>

  </section>

  <section>
    <h2>The module system</h2>

<pre><code data-trim contenteditable>
// file path: ./models/user.js
export class User { ... };

// file path: ./config/user.js
export let age = 24;
export const name = 'Tom';

// file path: ./utils.js
export default function toArray() { ... }

// file path: ./index.js
import { User } from './models/user';
import { age, name } from './config/user';
import toArray from './utils';
</code></pre>

  </section>

  <section>
    <h2>Destructuring for objects</h2>

<pre><code data-trim contenteditable>
var user = {
  age: 24,
  name: 'Tom',
  company: 'Blaaast'
};

// ES5 way of getting an object's properties
var age = user.age;
var name = user.name;

// ES6 way of getting an object's properties
const { age, name } = user;
</code></pre>

  </section>

  <section>
    <h2>Destructuring for arrays</h2>

<pre><code data-trim contenteditable>
var usersAges = [24, 32, 54, 16, 42];

// ES5 way of getting an array's elements
var userOneAge = usersAges[0];
var userFourAge = usersAges[3];
var lastUserAge = usersAges[usersAges.length - 1];

// ES6 way of getting an array's elements
const [userOneAge, , ,userFourAge] = usersAges;
</code></pre>

  </section>


  <section>
    <h2>Class</h2>

<pre><code data-trim contenteditable>
// ES6 classes
class User {
  constructor() {
    this.age = 24;
    this.name = 'Tom';
  }
  heresMyName() {
    return this.name;
  }
  get whatsMyName() {
    return this.name;
  }
  set whatsMyName(newName) {
    this.name = newName;
  }
  static hereIsAName() {

  }
  name
}

// extending classes in ES6
class Player extends User {
  constructor() {
    this.team = 'Yankee'; // throw error
    super(...arguments);
    this.team = 'Yankee'; // 'this' is available after 'super' call on extended classes
  }
  heresMyName() {
    super(...arguments);
    // Additional code specific for the player class
  }
}
</code></pre>

  </section>

  <section>

    <h2>Decorators</h2>

<pre><code data-trim contenteditable>
class User {
  @readOnly
  fullName() { return `${this.first} ${this.last}` }
}

// Installing the property
Object.defineProperty(Person.prototype, 'name', {
  value: specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
});

// How it roughly works
function readOnly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

let descriptor = readOnly(User.prototype, 'jobsByType', {
  enumerable: false,
  writable: true,
  configurable: false,
  value: null
});
Object.defineProperty(User.prototype, 'jobsByType', descriptor);

// Can take params and be applied to a class
@isTestable(true)
class MyClass { ... }
</code></pre>

  </section>

  <section>

    <h2>Async functions</h2>

<pre><code data-trim contenteditable>
async function() {
  this.set('isLoading', true);

  try {
    return await save();
  } catch {
    // Called on error
  } finally {
    // Always called once everything is done
    this.set('isLoading', false);
  }
}
</code></pre>

  </section>

  <section>

    <h2>Going beyond</h2>

    <p>for..of, map, weakmap, set, weakset, generator (yield), proxies...</p>

  </section>

</section>

<section>

  <section>
    <h1>
      A practical Ember example
      <small>Ember jobs</small>
    </h1>
  </section>

  <section>
    <h2>
      Destructuring
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
const { inject, computed } = Ember;
// Is equivalent to
const inject = Ember.inject;
const computed = Ember.computed;

// What does that mean ?
import computed, { readOnly } from 'ember-computed-decorators';
// Computed => default export
// Readonly => named export from 'ember-computed-decorators'
</code></pre>

    </p>
  </section>

  <section>
    <h2>
      New objects literal notation
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
// file path: ./app/controllers/index.js
import Ember from 'ember';

export default Ember.Controller.extend({
  ...

  actions: {
    jobTypeChanged(type) {
      this.set('type', type);
    },

    searchChanged(search) {
      this.set('search', search);
    }
  }
});
</code></pre>

    </p>
  </section>


  <section>
    <h2>
      Async functions
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
// file path: ./app/pods/new-position-form/component.js
export default Ember.Component.extend({
  ...

  actions: {

    async save() {
      try {
        let company = await company.save();

        return await store.createRecord('job', jobAttributes).save();
      }
    }

  }
}
</code></pre>

    </p>
  </section>


  <section>
    <h2>
      Testing (before Babel)
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
// file path: ./tests/acceptance/new-job-test.js
test('visiting / and opening/closing the  new job modal', function test(assert) {
  server.get('/jobs',      json(200, { jobs:      [] }));
  server.get('/companies', json(200, { companies: [] }));

  visit('/');

  click('#post-job');

  andThen(function() {
    isVisible(assert, '.new-job-modal');
  });

  click('.our-modal');

  andThen(function() {
    isNotVisible(assert, '.new-job-modal');
  })
});
</code></pre>

    </p>
  </section>

  <section>
    <h2>
      Testing (after Babel)
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
// file path: ./tests/acceptance/new-job-test.js
test('visiting / and opening/closing the  new job modal', async function test(assert) {
  server.get('/jobs',      json(200, { jobs:      [] }));
  server.get('/companies', json(200, { companies: [] }));

  await visit('/');
  await click('#post-job');

  isVisible(assert, '.new-job-modal');

  await click('.our-modal');

  isNotVisible(assert, '.new-job-modal');
});
</code></pre>

    </p>
  </section>

  <section>
    <h2>
      Using ES7 decorators
    </h2>

    <p>

<pre><code data-trim contenteditable style="max-height: 600px;">
filteredJobs: Ember.computed('jobsByType.[]', function() { ... });

@computed('jobsByType.[]')
filteredJobs(jobs) {
  ...
}

@computed('type', 'search', 'model.@each.{title,description,type,location,company,name}')
@readOnly
jobsByType(type, search) {
  ...
}

firstName: Ember.computed.alias('first')

@alias('first') firstName,
@empty('first') missingFirstName,
@match('first', /rob/i) isRob

// Ember data models
export default DS.Model.extend({
  @attr({ defaultTo: 'smith' }) name,
  @attr logo,
  @attr url,
  @attr jobs,

  @hasMany accounts,
  @belongsTo human

});
</code></pre>

    </p>
  </section>

</section>


<section>

  <h1>Conclusion</h1>
  <h2>How is Ember moving to the new JavaScript ? <small>(and features you should be aware of)</small></h2>
  <p>Transpilation ES6/7 => ES5 (and ES3 on activation)</p>
  <p>Addons for future ES7 not out-of-the-transpiler implementations</p>

</section>


<section>
  <h1>Questions ?</h1>
</section>
