# Anatomy
In this chapter we will learn about the main components of Ember CLI.

Ember CLI is a **Node.js** command line application that sits on top of
other libraries.

Its main component is **Broccoli**, a builder designed to keep builds as
fast as possible.

When we run `ember server`, **Broccoli** compiles our project and puts it
in a directory where it can be served using **Express.js**[^express], a **Node.js** library. **Express** not only serves
files but also extends Ember CLI functionality using its
**middlewares**. An example of this is **http-proxy**, which supports
the `--proxy` option that allows us to develop against our development
backend.

Out of the box, testing is powered by **QUnit** and **Testem**. By navigating to
**http:/localhost:4200/tests**, our tests run automatically.
We can also run Testem in **CI** or `--development` mode with the **ember
test** command. We can also use other testing frameworks like Mocha thanks to the `ember-cli-mocha` addon.

Ember CLI allows us to write our application using **ES6 Modules**. The
code is then transpiled (compiled)[^transpiled] to **AMD**[^amd] and
finally loaded with the minimalist **AMD** loader, **loader.js**.

We can use **CoffeeScript** if we want, but it is recommended to use
plain JS and ES6 modules where possible. In subsequent chapters, we'll
explore its syntax and features. Ember CLI ships with Babel[^babel] support,
which allows us to use next generation JavaScript without extra work.

Finally, we need to cover plugins that enhance the functionality of **Broccoli**. Each transformation your
files go through is done with a **Broccoli** plugin, e.g.
transpiling, minifying, finger-printing, uglifying. You can have your
own **Broccoli** plugins and plug them wherever you like throughout the build
process.

[^express]: [http://expressjs.com/](http://expressjs.com/)
[^babel]: [Babel](https://babeljs.io/)
[^transpiled]: The transpiling process is done with [es6-module-transpiler](https://github.com/esnext/es6-module-transpiler).
[^amd]: To know more about **AMD** checkout [their wiki](https://github.com/amdjs/amdjs-api/wiki/AMD)
