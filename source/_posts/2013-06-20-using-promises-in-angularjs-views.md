---
layout: post
title: "Using Promises in AngularJS Views"
date: 2013-06-20 08:32
comments: true
categories: 
---

One of the lesser known yet more surprisingly powerful features of [AngularJS](http://angularjs.org/) is the way in which it allows [promises](http://wiki.commonjs.org/wiki/Promises/A) to be used directly inside views.

To better understand the benefits of this feature, we'll first migrate a typical callback-style service to a promise-based interface.

## Working with callbacks

For now we'll sidestep a discussion on the advantages of promises compared to callbacks, and focus soley on their mechanics.

As a working example, let's take a look at an example service with a single *'getMessages'* function.

```js
var myModule = angular.module('myModule', []);
// From this point on, we'll attach everything to 'myModule'

myModule.factory('HelloWorld', function($timeout) {

    var getMessages = function(callback) {
      $timeout(function() {
        callback(['Hello', 'world!']);
      }, 2000);
    };

    return {
      getMessages: getMessages
    };

  });
```

This admittedly contrived service's *'getMessages'* function takes a callback, then waits two seconds (using Angular's [$timeout service](http://docs.angularjs.org/api/ng.$timeout)) before passing an array of messages to the callback function.

If we use this example service inside a controller, it looks like this:

```js
myModule.controller('HelloCtrl', function($scope, HelloWorld) {

    HelloWorld.getMessages(function(messages) {
      $scope.messages = messages;
    });

});
```

## Upgrading to promises

If we instead wanted our *'HelloWorld'* service to expose a promise-based API, it would look something like this.

```js
myModule.factory('HelloWorld', function($q, $timeout) {

  var getMessages = function() {
    var deferred = $q.defer();

    $timeout(function() {
      deferred.resolve(['Hello', 'world!']);
    }, 2000);

    return deferred.promise;
  };

  return {
    getMessages: getMessages
  };

});
```

You'll notice that we're now relying on Angular's [$q service](http://docs.angularjs.org/api/ng.$q) (based on [Kris Kowal's Q](https://github.com/kriskowal/q)) to create a [*'deferred'*](https://github.com/kriskowal/q#using-deferreds). We return the deferred's *'promise'* property as a public hook to its state, which is safely tucked away inside a closure.

Now that we have a promise API, we need to update the service interaction inside our controller.

```js
myModule.controller('HelloCtrl', function($scope, HelloWorld) {

    HelloWorld.getMessages().then(function(messages) {
      $scope.messages = messages;
    });

});
```

Our controller is essentially the same, except *'getMessages'* no longer accepts a callback. Instead, it takes no arguments, and returns a promise object.

As is standard for promises, it has a 'then' function that takes two arguments: a success callback and an error callback. For our purposes, we'll ignore the powerful error handling capabilities that promises afford to asyncronous code.

## Wiring up the view

In both the callback and promise versions of our controller, we end up with a *'messages'* property on the scope. Which means, of course, that our view would remain unchanged.

A very simple view that only displays the messages would look like this:

```html
<body ng-app="myModule" ng-controller="HelloCtrl">
  <h1>Messages</h1>
  <ul>
    <li ng-repeat="message in messages">{{ message }}</li>
  </ul>
</body>
```

In this case, we're simply iterating over the messages that were returned from the *'HelloWorld'* service.

## Using promises directly in the view

AngularJS allows us to streamline our controller logic by placing a promise directly on the scope, rather than manually handing the resolved value in a success callback.

Our original controller logic for handling the promise was relatively verbose, considering how simple the operation is:

```js
// BEFORE:

myModule.controller('HelloCtrl', function($scope, HelloWorld) {

  HelloWorld.getMessages().then(function(messages) {
    $scope.messages = messages;
  });

});
```

To simplify this, we can place the promise returned by *'getMessages'* on the scope:

```js
// AFTER:

myModule.controller('HelloCtrl', function($scope, HelloWorld) {

  $scope.messages = HelloWorld.getMessages();

});
```

When Angular encounters a promise inside the view, it automatically sets up a success callback and substitutes the promise for the resulting value once it has been resolved.

## Seeing it for yourself

I've set up a [plunk of this simple example](http://plnkr.co/edit/QBAB0usWXc96TnxqKhuA) so you can get a better feel for how this pattern works.

Using promises directly on the view is a feature of AngularJS that many people are unaware of, but it opens up a lot of opportunities for making our controllers as lean as possible.

In fact, once you've become familiar with the concept, you might be surprised how often you see that this pattern can be applied.