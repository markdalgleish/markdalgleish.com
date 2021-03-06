---
layout: post
title: "Testing jQuery Plugins Cross-Version with Grunt"
date: 2013-01-17 13:17
comments: true
categories: 
---

The [jQuery](http://jquery.com/) team have made the tough, but inevitable decision to [stop supporting IE8 and below as of jQuery v2.0](http://blog.jquery.com/2013/01/15/jquery-1-9-final-jquery-2-0-beta-migrate-final-released/), while maintaining v1.9 as the backwards compatible version for the forseeable future.

In the world of modern, evergreen and mobile browsers, this was a necessary move to ensure jQuery stays relevant. Of course, this split leaves plugin authors with a bit more responsibility.

Where previously we could simply require the most recent version of jQuery, we are now likely to want to support both 1.9.x and 2.x, allowing our plugins to work everywhere from IE6 to the most bleeding edge browsers.

To facilitate this, we'll run through the creation of a plugin using the popular JavaScript build tool, [Grunt](http://gruntjs.com/). We'll then configure our unit tests to run automatically across multiple versions of jQuery.

## A simple jQuery plugin

*Note: If you have an existing plugin that doesn't use Grunt, I'd suggest running through these steps in a clean directory and porting the resultant code into your project (with some manual tweaks, of course).*

Assuming you already have [Git](http://git-scm.com/) and [Node.js](http://nodejs.org/), we first need [Grunt-init](https://github.com/gruntjs/grunt-init) and the [Grunt command line interface](https://github.com/gruntjs/grunt-cli) installed globally. Run the following command to ensure you have the latest version:

```bash
$ npm install -g grunt-init grunt-cli
```

*Note: If you already have an older version of Grunt installed, you'll need to first remove it with `npm uninstall -g grunt`.*

We also need to install the [*'grunt-init-jquery'*](https://github.com/gruntjs/grunt-init-jquery) template into our *'~/.grunt-init'* directory by cloning the repository:

```bash
git clone git@github.com:gruntjs/grunt-init-jquery.git ~/.grunt-init/jquery
```

We can now scaffold a new jQuery project:

```bash
$ mkdir jquery.plugin
$ cd jquery.plugin
$ grunt-init jquery
```

Once we've responded to all the prompts, we're left with a basic jQuery plugin with [QUnit](http://qunitjs.com/) tests.

Before we continue, we need to install our Node dependencies by running the following command from within our new plugin directory:

```bash
$ npm install
```

We can run our placeholder tests like so:

```bash
$ grunt qunit

  Running "qunit:files" (qunit) task
  Testing test/plugin.html....OK
  >> 5 assertions passed (51ms)

  Done, without errors.
```

For the purposes of this tutorial, we're not terribly interested in the contents of the plugin. Instead, we'll focus solely on the build and test infrastructure.

Before we make changes to our placeholder project, it's worth having a closer look at what has been generated.

## Inspecting the build

All of the configuration for our Grunt build process sits inside our Gruntfile *(Gruntfile.js)* in our project directory.

We have *'qunit'* configuration, which looks for all QUnit files in the *'test'* directory:

```js
// snip...

qunit: {
  files: ['test/**/*.html']
},

// snip...
```

At the end of our Grunt configuration is the definition of our default task:

```js
// Default task.
grunt.registerTask('default', ['jshint', 'qunit', 'clean', 'concat', 'uglify']);
```

The default task is run when the *'grunt'* command is executed without any arguments:

```bash
$ grunt
```

## Inspecting the test

The QUnit test for our plugin resides in *'test/plugin.html'*. Its default markup looks like this:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Plugin Test Suite</title>
  <!-- Load local jQuery. This can be overridden with a ?jquery=___ param. -->
  <script src="../libs/jquery-loader.js"></script>
  <!-- Load local QUnit. -->
  <link rel="stylesheet" href="../libs/qunit/qunit.css" media="screen">
  <script src="../libs/qunit/qunit.js"></script>
  <!-- Load local lib and tests. -->
  <script src="../src/plugin.js"></script>
  <script src="plugin_test.js"></script>
  <!-- Removing access to jQuery and $. But it'll still be available as _$, if
       you REALLY want to mess around with jQuery in the console. REMEMBER WE
       ARE TESTING A PLUGIN HERE, THIS HELPS ENSURE BEST PRACTICES. REALLY. -->
  <script>window._$ = jQuery.noConflict(true);</script>
</head>
<body>
  <div id="qunit"></div>
  <div id="qunit-fixture">
    <span>lame test markup</span>
    <span>normal test markup</span>
    <span>awesome test markup</span>
  </div>
</body>
</html>
```

This page is responsible for including jQuery, QUnit (both JavaScript and CSS), our plugin, and any helpers required. It also provides the markup needed for QUnit to generate an HTML report.

You'll notice, the first script file included is *'../libs/jquery-loader.js'*. If we look at the contents of that file, we find this:

```js
(function() {
  // Default to the local version.
  var path = '../libs/jquery/jquery.js';
  // Get any jquery=___ param from the query string.
  var jqversion = location.search.match(/[?&]jquery=(.*?)(?=&|$)/);
  // If a version was specified, use that version from code.jquery.com.
  if (jqversion) {
    path = 'http://code.jquery.com/jquery-' + jqversion[1] + '.js';
  }
  // This is the only time I'll ever use document.write, I promise!
  document.write('<script src="' + path + '"></script>');
}());
```

By including this script, we now have the ability to add *'?jquery=X.X.X'* to the query string, when viewing this page in the browser.

Doing this will cause a hosted version of our specified version of jQuery to be included in the page rather than the default version provided inside our project.

## Preparing the build

You might think that we could simply modify the QUnit file matcher in our Gruntfile to add a query string, but this won't work. Files must exist on the file system, and query strings aren't part of that vocabulary.

To automatically run our tests with different query strings, we first need to host our test on a local server.

Luckily, Grunt has an [officially-supported *'connect'* task](https://github.com/gruntjs/grunt-contrib-connect) which does the work for us by running a server using [Connect](http://www.senchalabs.org/connect/).

To install the *'grunt-contrib-connect'* Grunt plugin, we need to install it, and automatically save it as a development dependency in our [*'package.json'*](http://package.json.nodejitsu.com/) file:

```bash
$ npm install --save-dev grunt-contrib-connect
```

Before we can use this Grunt plugin, we need to register it with Grunt by adding the following line to our Gruntfile's *'loadNpmTasks'*:

```bash
grunt.loadNpmTasks('grunt-contrib-connect');
```

We can configure our server by adding the following task configuration to our Gruntfile:

```js
connect: {
  server: {
    options: {
      port: 8085 // This is a random port, feel free to change it.
    }
  }
},
```

If we modify our default task to first include our newly configured *'connect'* task, this server will start every time the default task is executed, and stopped when the build has completed:

```js
// Default task.
grunt.registerTask('default', ['connect', jshint', 'qunit', 'clean', 'concat', 'uglify']);
```

Since we want to be able to test our plugin without having to concatenate and minify it, I recommend adding the following *'test'* task:

```js
grunt.registerTask('test', ['connect', 'jshint', 'qunit']);
```

We can now lint and test our code from the command line like so:

```bash
$ grunt test
```

## Configuring the test URLs

So far we have a local Connect server running every time we trigger a build, and we have a *'test'* task which will run the server before linting our code and running our QUnit tests.

However, you'll find that we're still pointing QUnit at the file system. Instead, we want it to point to our new server.

To achieve this, we'll pass QUnit an array of URLs rather than files:

```js
qunit: {
  all: {
    options: {
      urls: [
        'http://localhost:<%= connect.server.options.port %>/test/plugin.html'
      ]
    }
  }
},
```

Now when we run our tests, we should basically see the same result as before:

```bash
$ grunt test

  Running "connect:server" (connect) task
  Starting connect web server on localhost:8085.

  Running "jshint:gruntfile" (jshint) task
  >> 1 file lint free.

  Running "jshint:src" (jshint) task
  >> 1 file lint free.

  Running "jshint:test" (jshint) task
  >> 1 file lint free.

  Running "qunit:all" (qunit) task
  Testing http://localhost:8085/test/plugin.html....OK
  >> 5 assertions passed (41ms)

  Done, without errors.
```

You'll notice that this time, QUnit is accessing a URL instead of a file. This means that we're now free to add query strings to our URLs, allowing us to automate testing across multiple versions of jQuery with ease:

```js
qunit: {
  all: {
    options: {
      urls: [
        'http://localhost:<%= connect.server.options.port %>/test/plugin.html?jquery=1.9.0',
        'http://localhost:<%= connect.server.options.port %>/test/plugin.html?jquery=2.0.0b1'
      ]
    }
  }
},
```

Since there will be a lot of repetition in the URLs, let's clean it up with use of the [Array prototype's 'map' method](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Array/map):

```js
qunit: {
  all: {
    options: {
      urls: ['1.9.0', '2.0.0b1'].map(function(version) {
        return 'http://localhost:<%= connect.server.options.port %>/test/plugin.html?jquery=' + version;
      })
    }
  }
},
```

If we run our tests, you'll see multiple URLs have been loaded, and twice as many assertions have passed:

```bash
$ grunt test

  Running "connect:server" (connect) task
  Starting connect web server on localhost:8085.

  Running "jshint:gruntfile" (jshint) task
  >> 1 file lint free.

  Running "jshint:src" (jshint) task
  >> 1 file lint free.

  Running "jshint:test" (jshint) task
  >> 1 file lint free.

  Running "qunit:all" (qunit) task
  Testing http://localhost:8085/test/plugin.html?jquery=1.9.0....OK
  Testing http://localhost:8085/test/plugin.html?jquery=2.0.0b1....OK
  >> 10 assertions passed (98ms)

  Done, without errors.
```

## Making it bulletproof

By default, this setup loads each version directly from the jQuery site. If you're anything like me, you sometimes develop with little to no internet connectivity, and this limitation would prevent you from running the full suite.

It's a good idea to add each major supported version of jQuery to your *'lib/jquery'* directory (with a 'jquery-x.x.x' naming convention), and modify *'libs/jquery-loader.js'* to load these local copies instead:

```js
(function() {
  // Default to the local version.
  var path = '../libs/jquery/jquery.js';
  // Get any jquery=___ param from the query string.
  var jqversion = location.search.match(/[?&]jquery=(.*?)(?=&|$)/);
  // If a version was specified, use that version from code.jquery.com.
  if (jqversion) {
    path = '../libs/jquery/jquery-' + jqversion[1] + '.js';
  }
  // This is the only time I'll ever use document.write, I promise!
  document.write('<script src="' + path + '"></script>');
}());
```

## Testing in the cloud

As always, it's a great idea to automatically run these tests after every push to [GitHub](https://github.com/), or on every pull request that is sent to you. To achieve this, we can leverage [Travis CI](http://travis-ci.org/) with only a couple of changes to our project.

First add the [*'.travis.yml'*](http://about.travis-ci.org/docs/user/languages/javascript-with-nodejs/) configuration file to your plugin's base directory:

```
language: node_js

node_js:
  - 0.8
```

Then, set the [*'npm test'*](https://npmjs.org/doc/test.html) script in your *'package.json'* file to run our new Grunt *'test'* task:

```js
// Snip...

"scripts": {
  "test": "grunt test"
},

// Snip...
```

Finally, follow the [official Travis CI guide](http://about.travis-ci.org/docs/user/getting-started/#Step-one%3A-Sign-in) to create an account, if needed, and activate the GitHub service hook. Once completed, you'll have the confidence of knowing that the downloadable version of your plugin can't be broken by mistake.

## Keeping it in check

Now that we have a framework for testing multiple versions, it's worth testing the minimum jQuery version your plugin supports, and each major version above it.

At a minimum, I'd recommend testing in 1.9.x and 2.x to ensure that any differences between the two versions don't inadvertently break your plugin. Since both versions will be developed in parallel as long as old versions of IE maintain significant market share, it's the least we can do for our users.

*Update (19 Feb 2013): This article now reflects changes made in Grunt v0.4*