SystemJS Build Tool
===

Provides a single-file build for SystemJS of mixed-dependency module trees.

Builds ES6 into ES5, CommonJS, AMD and globals into a single file in a way that supports the CSP SystemJS loader
as well as circular references.

Example
---

app.js
```javascript
import $ from "./jquery";
export var hello = 'es6';
```

jquery.js
```javascript
define(function() {
  return 'this is jquery';
});
```

Builds into:

```javascript
// Declarative System.register (ES6)
// System.register(name, deps, declare)
System.register('app', ['./jquery'], function(deps) {
  var $, hello;
  return {
    exports: {
      get hello() {
        return hello;
      },
      set hello(val) {
        hello = val;
      }
    },
    execute: function() {
      $ = deps[0]['default'];
      hello = 'es6';
    }
  }
});

define('jquery', function() {
  return 'this is jquery';
});
```

It also provides a dynamic System.register variation for CommonJS and Globals. For example, CommonJS is output as:

```javascript
// Dynamic module System.register
// System.register(name, deps, executingRequire, execute);
System.register("some/cjs", [], true, function(require, exports, __moduleName) {
  var global = System.global;
  var __define = global.define;
  global.define = undefined;
  var module = { exports: exports };
  var process = System.get("@@nodeProcess")['default'];
  exports.cjs = true;
  
  global.define = __define;
  return module.exports;
});
```

The `true` boolean argument in the above indicates that CommonJS requires are execution driving,
as opposed to AMD which delays execution until all dependencies have been executed.

Usage
---

### Install

```javascript
  npm install systemjs-builder
```

### Basic Use

```javascript
  var builder = require('systemjs-builder');

  builder.build('myModule', {
    baseURL: path.resolve('some/folder'),

    // any map config
    map: {
      jquery: 'jquery-1.2.3/jquery'
    },

    // etc. any SystemJS config
  }, 'outfile.js')
  .then(function() {
    console.log('Build complete');
  })
  .catch(function(err) {
    console.log('Build error');
    console.log(err);
  });
```

### Advanced build

The trace trees can be adjusted between tracing and building allowing for custom build layer creation.

Some simple trace tree operators are provided for subtraction addition and intersection.

In this example we build `app/core` excluding `app/corelibs`:

```javascript
  var builder = require('systemjs-builder');

  builder.config({
    baseURL: '...',
    map: {

    }, // etc. config
  });

  builder.trace('app/main')
  .then(function(appTree) {

    return builder.trace('app/corelibs')
    .then(function(coreTree) {
      return builder.subtractTrees(appTree, coreTree);
    });
  })
  .then(function(appMinusCoreTree) {
    return builder.buildTree(appMinusCoreTree, 'app/main', 'output-file.js');
  });
```

Additional tree functions include `addTrees` and `intersectTrees`.

License
---

MIT
