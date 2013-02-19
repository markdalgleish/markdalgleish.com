---
layout: post
title: "Test-Driven Node.js Development with Grunt"
date: 2012-09-12 13:04
comments: true
categories: 
---

One of the great things about working in a [Node.js](http://nodejs.org) environment is that it encourages you to break your work down into discrete modules. Separating your work into smaller files is a good first step, but publishing to [npm](http://npmjs.org) is so simple that creating small modules for others to share is a great way to give back to the community.

When writing small modules that adhere to the [Unix philosophy](http://en.wikipedia.org/wiki/Unix_philosophy) of small programs doing one thing well, writing in a [test-driven](http://en.wikipedia.org/wiki/Test-driven_development) manner is almost a no brainer. However, getting a new Node project up and running can be a tedious process, particularly if you're new to Node.

Luckily, [Ben Alman](http://benalman.com) has created [Grunt](http://gruntjs.com), a project which is fast becoming the build tool of choice for the JavaScript community.

## Getting started with Grunt

Grunt is pitched as a replacement for the usual [Make](http://www.gnu.org/software/make/)/[Jake](https://github.com/mde/jake)/[Cake](http://coffeescript.org/documentation/docs/cake.html) style of tool by encouraging the use and development of [plugins](http://gruntjs.com).

A handful of plugins are officially maintained by the Grunt team, but it's the [Grunt-init task](https://github.com/gruntjs/grunt-init) that is the most powerful for those just starting out with Grunt. You can use this task to create a new Gruntfile, or to generate scaffolding for jQuery, [CommonJS](http://www.commonjs.org/) and Node.js projects.

To get started (assuming you have [Git](http://git-scm.com/) and Node.js installed), first install Grunt:

``` bash
npm install -g grunt-init grunt-cli
```

This will install the Grunt CLI tool and Grunt-init globally, exposing the 'grunt' and 'grunt-init' commands respectively.

To complete our setup, we need to install the [Grunt-init Node template](https://github.com/gruntjs/grunt-init-node) into our *'~/.grunt-init'* directory using Git:

``` bash
$ git clone git@github.com:gruntjs/grunt-init-node.git ~/.grunt-init/node
```

## Scaffolding our module

For our test-driving example, we'll create a simple project called 'Palindrode' for detecting whether a string is a [palindrome](http://en.wikipedia.org/wiki/Palindrome). Our project will expose a single function called 'test' which, similar to the 'test' method of the 'RegExp' prototype, returns a boolean value.

To begin, we'll initialise a new Node project:

``` bash
$ mkdir palindrode
$ cd palindrode
$ grunt-init node
```

At this point, Grunt-init will ask a series of questions, most of which have sensible defaults based on your system. Grunt-init even detects that our module is called 'palindrode' based on the current directory. Upon answering them, the scaffolding for our new project is created:

``` bash
Writing .gitignore...OK
Writing .jshintrc...OK
Writing Gruntfile.js...OK
Writing README.md...OK
Writing lib/palindrode.js...OK
Writing test/palindrode_test.js...OK
Writing LICENSE-MIT...OK
Writing package.json...OK

Initialized from template "node".
You should now install project dependencies with npm install. After that, you
may execute project tasks with grunt. For more information about installing
and configuring Grunt, please see the Getting Started guide:

http://gruntjs.com/getting-started

Done, without errors.
```

As the instructions have just informed us, we need to install our Node dependencies:

``` bash
$ npm install
```

The code for our module goes in the newly generated *lib/palindrode.js*, and the tests go in *test/palindrode_test.js*.

Just to get used to the testing workflow, we can test our sample code:

``` bash
$ grunt nodeunit
Running "nodeunit:files" (nodeunit) task
Testing palindrode_test.js.OK
>> 1 assertions passed (4ms)

Done, without errors.
```

Currently there is only one test, and the placeholder code to make the test pass, so you should find this passes with no issues.

## Going for a test drive

Our test file, *test/palindrode_test.js*, already contains a placeholder test, including a Nodeunit reference:

``` js
'use strict';

var palindrode = require('../lib/palindrode.js');

/*
  ======== A Handy Little Nodeunit Reference ========
  https://github.com/caolan/nodeunit

  Test methods:
    test.expect(numAssertions)
    test.done()
  Test assertions:
    test.ok(value, [message])
    test.equal(actual, expected, [message])
    test.notEqual(actual, expected, [message])
    test.deepEqual(actual, expected, [message])
    test.notDeepEqual(actual, expected, [message])
    test.strictEqual(actual, expected, [message])
    test.notStrictEqual(actual, expected, [message])
    test.throws(block, [error], [message])
    test.doesNotThrow(block, [error], [message])
    test.ifError(value)
*/

exports['awesome'] = {
  setUp: function(done) {
    // setup here
    done();
  },
  'no args': function(test) {
    test.expect(1);
    // tests here
    test.equal(palindrode.awesome(), 'awesome', 'should be awesome.');
    test.done();
  },
};

```

Let's strip this back to a couple of simple palindrome tests:

``` js
'use strict';

var palindrode = require('../lib/palindrode.js');

exports['test'] = {
  'correctly matches palindrome string': function(test) {
    test.expect(1);
    test.ok(palindrode.test('Was it a car or a cat I saw?'));
    test.done();
  },
  'correctly matches non-palindrome strings': function(test) {
    test.expect(1);
    test.equal(palindrode.test('This is not a palindrome'), false);
    test.done();
  }
};
```

If we run the tests now, you'll find they fail:

``` bash
$ grunt nodeunit
Running "nodeunit:files" (nodeunit) task
...
Warning: 4/4 assertions failed (13ms) Use --force to continue.

Aborted due to warnings.
```

To make our tests pass, we need to edit our *lib/palindrode.js*, which currently looks like this:

``` js
/*
 * palindrode
 * https://github.com/markdalgleish/palindrode
 *
 * Copyright (c) 2013 Mark Dalgleish
 * Licensed under the MIT license.
 */

'use strict';

exports.awesome = function() {
  return 'awesome';
};
```

We need to replace the placeholder *awesome* function with a *test* function which returns a boolean. The important thing to note is that our module must ignore spaces and punctuation in order to correctly match sentence palindromes.

Let's write a function which does this:

``` js
exports.test = function(string) {
  string = string.match(/[a-z0-9]/gi).join('').toLowerCase();
  return string === string.split('').reverse().join('');
};
```

This is enough to pass our two tests:

``` bash
$ grunt nodeunit
Running "nodeunit:files" (nodeunit) task
Testing palindrode_test.js..OK
>> 2 assertions passed (6ms)

Done, without errors.
```

Currently our function will return true when passed empty strings or strings containing only spaces/punctuation. It will also throw an error if we pass it anything other than a string (or, more precisely, anything that doesn't have a *match* method).

Let's write a few more tests to make sure that our function is a bit more resilient:

``` js
'returns false when passed blank strings': function(test) {
  test.expect(1);
  test.equal(palindrode.test(''), false);
  test.done();
},
'returns false when passed only punctuation/spaces': function(test) {
  test.expect(1);
  test.equal(palindrode.test(' ?., '), false);
  test.done();
},
'returns false when passed non-string values': function(test) {
  test.expect(2);
  test.equal(palindrode.test(1234), false, 'should not accept numbers');
  test.equal(palindrode.test(), false, 'should not accept undefined');
  test.done();
}
```

Of course, these new tests won't pass:

``` bash
$ grunt nodeunit
Running "nodeunit:files" (nodeunit) task
...
Warning: 6/8 assertions failed (30ms) Use --force to continue.
```

Let's upgrade Palindrode's *test* function to pass these new tests:

``` js
exports.test = function(string) {
  if (typeof string !== 'string') {
    return false;
  }
  var matches = string.match(/[a-z0-9]/gi);
  if (matches === null) {
    return false;
  }
  string = matches.join('').toLowerCase();
  return string.length > 0 && string === string.split('').reverse().join('');
};
```

Our new tests now pass with flying colours:

``` bash
$ grunt nodeunit
Running "nodeunit:files" (nodeunit) task
Testing palindrode_test.js.....OK
>> 6 assertions passed (6ms)

Done, without errors.
```

In the spirit of *red/green/refactor*, our final step is to clean up our code. We'll move variable declarations to the top of the scope, use more descriptive variable names and separate the more complicated logic:

``` js
exports.test = function(string) {
  var relevantCharacters,
      filteredString,
      reversedString;

  if (typeof string !== 'string') {
    return false;
  }

  relevantCharacters = string.match(/[a-z0-9]/gi);

  if (!relevantCharacters) {
    return false;
  }

  filteredString = relevantCharacters.join('').toLowerCase();
  reversedString = filteredString.split('').reverse().join('');

  return filteredString.length > 0 && filteredString === reversedString;
};
```

With Grunt's help, writing a lightweight, tested module is incredibly easy.

It's just as easy to include more tasks in our build to improve the quality of our module. For example, we can run our code through [JSHint](http://www.jshint.com) with the built-in *jshint* task. By default, if a Gruntfile exists in the current directory, running *grunt* with no parameters runs the *jshint* and *nodeunit* tasks.

If we wanted, the code is also ready to be published to npm using the *npm publish* command.

## Testing continuously

We now have some tests and the code to make it pass. Thanks to Grunt, we also have a *package.json* file which specifies all our dependencies, and indicates that the [*npm test*](https://npmjs.org/doc/test.html) script should run *grunt nodeunit*.

All of this allows us to take advantage of [Travis CI](http://travis-ci.org/), a web-based continuous integration service.

There's a few things we need to do to make this happen.

First, our code needs to be hosted on [GitHub](https://www.github.com).

Next we create a file called *.travis.yml*, and specify that this is a Node.js project:

```
language: node_js

node_js:
  - 0.8
```

The final step is to log in to [Travis CI](http://travis-ci.org/) using our GitHub account. On your Travis profile page you will see a list of public repositories, each with a switch to activate GitHub's service hook.

Once activated, any time you push module updates to GitHub, the *npm test* script, which in turn calls onto *grunt nodeunit*, will be run in the cloud. If the build fails, we get notified via email.

## Moving forward

With all of this in place, we now have a solid, tested module. Maintaining the quality of your module is simple, and thanks to GitHub's new [Commit Status API](https://github.com/blog/1227-commit-status-api), Travis CI can now [run your test suite against all pull requests](http://about.travis-ci.org/blog/2012-09-04-pull-requests-just-got-even-more-awesome/).

To see a simple example of this setup in one of my own projects, check out [lanyrd-scraper](https://github.com/markdalgleish/node-lanyrd-scraper), a Node module for loading event data from [Lanyrd.com](http://lanyrd.com).

*Update (19 Feb 2013): This article now reflects changes made in Grunt v0.4*