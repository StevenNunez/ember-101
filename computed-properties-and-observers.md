# Computed Properties and Observers

We already covered computed properties and use them in different parts
of our applications, one of them was on the friend model:

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  // ...
  fullName: Ember.computed('firstName', 'lastName', function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  })
});
~~~~~~~~

With the code above we created a new property on the model called
**fullName** which depends on **firstName** and **lastName**, the
computed properties will be called once at the beginning and then the result will be
cached until any of the dependent properties change.

Next we'll like to talk about a couple of features and things to keep
in mind when defining computed properties.

## An alternative syntax for computed properties

Ember extends the **function** prototype with the function
**property** to allow us to specify computed properties in a
different way, instead of using `Ember.computed` we can specify a
computed property like the following:

{title="CP in app/models/friend.js via .property function", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  // ...
  fullName: function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName');
});
~~~~~~~~

We achieve the same result using the `prototype` extension or `Ember.computed`, what to use is
just a matter of taste, some people like `Ember.computed` and others
`.property`.

We need to keep in mind though that we can use `.property` only
because prototype extensions are enabled by default, if we decide to
turn them off then we wouldn't be able to use this functionality.

I> The Ember.js guides have a section about disabling prototype
I> extensions, if we are thinking about turning them off we should give
I> it a read and understand the implications: [Disabling prototype extensions](http://emberjs.com/guides/configuring-ember/disabling-prototype-extensions).

## Computed Property function signature

The functions we have been using to declare a computed property have
looked like the following:

{title="Computed Property Function", lang="JavaScript"}
~~~~~~~~
fullName: function() {
  return this.get('firstName') + ' ' + this.get('lastName');
}.property('firstName', 'lastName');
~~~~~~~~

Using the previous signature in the **function** we passed to
`Ember.computed` or `.property` we get computed properties working,
but we can optionally specify it like the following:

{title="Computed Property Function", lang="JavaScript"}
~~~~~~~~
fullName: function(key, value, oldValue) {
  return this.get('firstName') + ' ' + this.get('lastName');
}.property('firstName', 'lastName');
~~~~~~~~

With that we can add support for setting the value of a computed
property and handle  how it should behave, the following is extracted
from the Ember.js documentation where they use **firstName** and
**lastName** too:

{title="Computed Property with set support, lang="JavaScript"}
~~~~~~~~
fullName: function(key, value, oldValue) {
  if (arguments.length === 1) {
    //
    // Works as getter
    //

    return this.get('firstName') + ' ' + this.get('lastName')
  } else {

    //
    // Works as setter
    //

    var name = value.split(' ');

    this.set('firstName', name[0]);
    this.set('lastName', name[1]);

    return value;
  }
}.property('firstName', 'lastName')
~~~~~~~~

I> For the curious the following class has the implementation for [computed property](https://github.com/emberjs/ember.js/blob/v1.7.0/packages/ember-metal/lib/computed.js#L78).


Why did we avoid mentioning that we can use a computed property as
setter? The reason is that this is a very uncommon scenario which tends to
cause a lot of confusion on people, ideally we should use computed
properties as Read-Only. In a later version of Ember this might be the
default, there is an issue created by
[Stefan Penner](https://twitter.com/stefanpenner) which aims to make
computed properties Read-Only by default: [default readOnly
CP #9290](https://github.com/emberjs/ember.js/issues/9290).


## Computed Properties gotchas

Computed properties and observers are normally fired whenever we call
set on the property they depend on, the downside of this is that they
will be recalculated even if the value is the same, also if we are
using version 1.7 of Ember there are some bugs that cause computed
properties and observers to misbehave under some circumstances.

Some of the bugs have been fixed in  the upcoming version of
Ember.js (1.8) but computed properties and observes are still being
called even if the property didn't change.


Fortunately for us, [Gavin Joyce](https://twitter.com/gavinjoyce)
wrote an **ember-cli-addon** called
[ember-computed-change-gate](https://github.com/GavinJoyce/ember-computed-change-gate)
which offers an alternative function to define computed properties
and fixes observers such that they'll be only called if the property
they depend on have changed.

We can install the addon with `npm i ember-computed-change-gate
--save-dev` and use it in our friends model like the following:

{title="Using ember-computed-change-gate in app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';
import changeGate from 'ember-computed-change-gate/change-gate';

export default DS.Model.extend({
  // ...
  fullName: changeGate('firstName', 'lastName', function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  })
});
~~~~~~~~

With that our computed property will be called only when the value of
the dependent key has actually changed to a different value.

## Observers