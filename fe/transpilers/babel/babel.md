---
tags:
  - babel
  - ECMAScript
---

Introduction
Babel is a toolchain/transpiler mainly used to convert ECMAScript 2015+ code into a backwards compatible version of JavaScript in current and older browsers and environments.

Features:
- Transform syntax.
- Polyfill missing features in the target environment.
- Source code transformation.
- More.

Babel modules
All Babel modules are published as separated npm packages scoped under `@babel`. Main ones:
- `@babel/core`: can `require` directly in your Javascript project.
- `@babel/cli`: a tool that allows you to use babel from the terminal.

Plugins
Plugins are small Javascript programs that instruct Babel on how to tranform your code (e.g. `@babel/plugin-transform-arrow-functions`: transform ES2015+ arrow syntax to ES5).

You can even write your own plugins.

Preset
A preset is a pre-determined set of plugins (e.g. `@babel/preset-env`). Just like plugins, you can create your own presets to create any plugins combination you need.