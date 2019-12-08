rollup-plugin-external-globals
==============================

[![Build Status](https://travis-ci.org/eight04/rollup-plugin-external-globals.svg?branch=master)](https://travis-ci.org/eight04/rollup-plugin-external-globals)
[![codecov](https://codecov.io/gh/eight04/rollup-plugin-external-globals/branch/master/graph/badge.svg)](https://codecov.io/gh/eight04/rollup-plugin-external-globals)
[![install size](https://packagephobia.now.sh/badge?p=rollup-plugin-external-globals)](https://packagephobia.now.sh/result?p=rollup-plugin-external-globals)

Transform external imports into global variables like Rollup's `output.globals` option. See [rollup/rollup#2374](https://github.com/rollup/rollup/issues/2374)

Installation
------------

```
npm install -D rollup-plugin-external-globals
```

Usage
-----

```js
import externalGlobals from "rollup-plugin-external-globals";

export default {
  input: ["entry.js"],
  output: {
    dir: "dist",
    format: "es"
  },
  plugins: [
    externalGlobals({
      jquery: "$"
    })
  ]
};
```

The above config transforms

```js
import jq from "jquery";

console.log(jq(".test"));
```

into

```js
console.log($(".test"));
```

It also transforms dynamic import:

```js
import("jquery")
  .then($ => {
    $ = $.default || $;
    console.log($(".test"));
  });

// transformed
Promise.resolve($)
  .then($ => {
    $ = $.default || $;
    console.log($(".test"));
  });
```

> **Note:** when using dynamic import, you should notice that in ES module, the resolved object is aways a module namespace, but the global variable might be not.

> **Note:** this plugin only works with import/export syntax. If you are using a module loader transformer e.g. `rollup-plugin-commonjs`, you have to put this plugin *after* the transformer plugin.

API
----

This module exports a single function.

### createPlugin

```js
const plugin = createPlugin(
  globals: Object | Function,
  {
    include?: Array,
    exclude?: Array,
    dynamicWrapper?: Function
  } = {}
);
```

`globals` is a `moduleId`/`variableName` map. For example, to map `jquery` module to `$`:

```js
const globals = {
  jquery: "$"
}
```

or provide a function that takes the `moduleId` and returns the `variableName`.

```js
const globals = (id) => {
  if (id === "jquery") {
    return "$";
  }
}
```

`include` is an array of glob patterns. If defined, only matched files would be transformed.

`exclude` is an array of glob patterns. Matched files would not be transformed.

`dynamicWrapper` is used to specify dynamic imports. Below is the default.

```js
const dynamicWrapper = (id) => {
  return `Promise.resolve(${id})`;
}
```

Virtual modules are always transformed.

Changelog
---------

* 0.5.0 (Dec 8, 2019)

  - Add: `dynamicWrapper` option.
  - Add: now `globals` can be a function.
  - Bump dependencies/peer dependencies.

* 0.4.0 (Sep 24, 2019)

  - Add: transform dynamic imports i.e. `import("foo")` => `Promise.resolve(FOO)`.

* 0.3.1 (Jun 6, 2019)

  - Fix: all export-from statements are incorrectly transformed.
  - Bump dependencies.

* 0.3.0 (Mar 25, 2019)

  - Fix: temporary variable name conflicts.
  - **Breaking: transform virtual modules.** Now the plugin transforms proxy modules generated by commonjs plugin.
  - Bump dependencies.

* 0.2.1 (Oct 2, 2018)

  - Fix: don't skip export statement.

* 0.2.0 (Sep 12, 2018)

  - Change: use `transform` hook.
  - Add: rewrite conflicted variable names.
  - Add: handle export from.

* 0.1.0 (Aug 5, 2018)

  - Initial release.
