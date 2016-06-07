# Computed Properties and Observers

We already covered computed properties, which we use in different parts
of our applications. One of these uses occurs on the friend model:

{title="app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';

export default DS.Model.extend({
  // ...
  fullName: Ember.computed('firstName', 'lastName', {
    get() {
      return this.get('firstName') + ' ' + this.get('lastName');
    }
  })
});
~~~~~~~~

With the code above, we created a new property on the model called
**fullName** that depends on **firstName** and **lastName**. The
computed properties are called once at the beginning and the result is
cached until any of the dependent properties change.

Next we'll talk about a couple of features and things to keep
in mind when defining computed properties.

## Computed Property function signature

The functions we've used to declare a computed property have looked
like the following:

{title="Computed Property Function", lang="JavaScript"}
~~~~~~~~
fullName: Ember.computed('firstName', 'lastName', {
  get() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }
})
~~~~~~~~

We can also add support for setting the value of a computed
property and handling how it should behave. The following example
shows how:

{title="Computed Property with set support", lang="JavaScript"}
~~~~~~~~
fullName: Ember.computed('firstName', 'lastName', {
  get() {
    return this.get('firstName') + ' ' + this.get('lastName')
  },

  set(key, value) {
    var name = value.split(' ');

    this.set('firstName', name[0]);
    this.set('lastName', name[1]);

    return value;
  }
})
~~~~~~~~

I> For the curious, the following class has the implementation for [computed property](https://github.com/emberjs/ember.js/blob/v1.13.4/packages/ember-metal/lib/computed.js#L78).


Why didn't we mention that we can use a computed property as setter?
This is a very uncommon scenario that tends to cause a lot of
confusion for people. Ideally, we use computed properties as
Read-Only. In a later version of Ember, this might be the
default. [Stefan Penner](https://twitter.com/stefanpenner) created an
issue that aims to make computed properties Read-Only by default:
[default readOnly CP
#9290](https://github.com/emberjs/ember.js/issues/9290).


## Computed Properties gotchas

Computed properties and observers are normally fired whenever we call
`this.set()` on the property they depend on. The downside of this is that they
will be recalculated even if the value is the same.

Fortunately for us, [Gavin Joyce](https://twitter.com/gavinjoyce)
wrote an **ember-cli-addon** called
[ember-computed-change-gate](https://github.com/GavinJoyce/ember-computed-change-gate)
that offers an alternative function to define computed properties
and that fixes observers such that they are only called if the property
they depend on has changed.

We can install the addon with `npm i ember-computed-change-gate
--save-dev` and use it in our friends model like so:

{title="Using ember-computed-change-gate in app/models/friend.js", lang="JavaScript"}
~~~~~~~~
import DS from 'ember-data';
import Ember from 'ember';
import changeGate from 'ember-computed-change-gate/change-gate';

export default DS.Model.extend({
  //
  // Currently changeGate only support one property
  //
  capitalizedFirstName: changeGate('firstName', function(firstName) {
    return Ember.String.capitalize(firstName);
  })
});
~~~~~~~~

Now our computed property `capitalizedFirstName` will be called
only when the value of the dependent key has changed to a
different value.

## Observers

Ember has a built-in implementation of the
[Observer pattern](http://en.wikipedia.org/wiki/Observer_pattern),
which allows us to keep track of changes in any property or
computed property.

Let's use an observer on the article-row component to do something every time the state changes, let's add the following:

{title="app/components/loans/loan-row.js", lang="JavaScript"}
~~~~~~~~
  stateChanged: Ember.observer('loan.returned', function() {
    var loan = this.get('loan');
    console.log('OMG Expensive operation because loan state changed');
  }),
~~~~~~~~

We define an observer calling `Ember.observer` which receives any
number of properties to observe and the function to call when any of
the properties change.

I> We might find some examples where observers are set calling
I> `.observer('property')` at the end of a function definition.
I> This pattern is valid but it relies on a mechanism called prototype
I> extensions which might get removed in future versions of Ember.
I> Please refer to the following pull request for more information
I> [emberjs/guides/pull/110](https://github.com/emberjs/guides/pull/110)


We can also create an observer using `addObserver` from
[Ember.Observable](http://emberjs.com/api/classes/Ember.Observable.html).
We could define the `stateChanged` observer like this:

{title=""app/components/loans/loan-row.js", lang="JavaScript"}
~~~~~~~~
  setObserver: function() {
    this.addObserver('loan.returned', this, this.stateChanged);
  }.on('init'),
  stateChanged() {
    var loan = this.get('loan');
    console.log('OMG Expensive operation because article state changed');
  },
~~~~~~~~

## Observing collections

Ember adds two convenient properties to collections. We can use
them if we want to observe changes to any of the members' properties,
or if we want to do something every time an element is
added or removed.

The first property is
[.[]](http://emberjs.com/api/classes/Ember.Array.html#property__),
which is just a special handler that changes every time the
collection content changes.

The second one is
[@each](http://emberjs.com/api/classes/Ember.Array.html#property__each),
which allows us to observe properties on each of the items in the
collection.

We can use the previous function in our articles index to call a
function when we add a new article, and then other one when we change
the state of an article:


{title="app/controllers/loans/index.js", lang="JavaScript"}
~~~~~~~~
import Ember from 'ember';

export default Ember.Controller.extend({
  contentDidChange: Ember.observer('model.[]', function() {
    console.log('Called when we add or removed a loan.');
  }),
  stateDidChange: Ember.observer('model.@each.returned, function() {
    console.log('Called when the state property change for any of the articles.');
  })
});
~~~~~~~~

If we visit any of our friends' profiles and change the state for any
loan or add a new one, we'll see the relevant messages in the
browser's console.
