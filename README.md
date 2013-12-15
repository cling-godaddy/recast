# recast, _v_. [![Build Status](https://travis-ci.org/benjamn/recast.png?branch=master)](https://travis-ci.org/benjamn/recast)
1. to give (a metal object) a different form by melting it down and reshaping it.
1. to form, fashion, or arrange again.
1. to remodel or reconstruct (a literary work, document, sentence, etc.).
1. to supply (a theater or opera work) with a new cast.

Installation
---

From NPM:

    npm install recast
    
From GitHub:

    cd path/to/node_modules
    git clone git://github.com/benjamn/recast.git
    cd recast
    npm install .

Usage
---

In less poetic terms, Recast exposes two essential interfaces, one for parsing JavaScript code (`require("recast").parse`) and the other for reprinting modified syntax trees (`require("recast").print`).

Here's a simple but non-trivial example of how you might use `.parse` and `.print`:
```js
var recast = require("recast");

// Let's turn this function declaration into a variable declaration.
var code = [
    "function add(a, b) {",
    "  return a +",
    "    // Weird formatting, huh?",
    "    b;",
    "}"
].join("\n");

// Parse the code using an interface similar to require("esprima").parse.
var ast = recast.parse(code);
```
Now do *whatever* you want to `ast`. Really, anything at all!
```js
// Grab a reference to the function declaration we just parsed.
var add = ast.program.body[0];

// Make sure it's a FunctionDeclaration (optional).
var n = recast.types.namedTypes;
n.FunctionDeclaration.assert(add);

// If you choose to use recast.builders to construct new AST nodes, all builder
// arguments will be dynamically type-checked against the Mozilla Parser API.
var b = recast.types.builders;

// This kind of manipulation should seem familiar if you've used Esprima or the
// Mozilla Parser API before.
ast.program.body[0] = b.variableDeclaration("var", [
    b.variableDeclarator(add.id, b.functionExpression(
        null, // Anonymize the function expression.
        add.params,
        add.body
    ))
]);

// Just for fun, because addition is commutative:
add.params.push(add.params.shift());
```
When you finish manipulating the AST, let `recast.print` work its magic:
```js
var output = recast.print(ast).code;
```
The `output` string now looks exactly like this, weird formatting and all:
```js
var add = function(b, a) {
  return a +
    // Weird formatting, huh?
    b;
}
```
The magic of Recast is that it reprints only those parts of the syntax tree that you modify. In other words, the following identity is guaranteed:
```js
recast.print(recast.parse(source)).code === source
```
Whenever Recast cannot reprint a modified node using the orginal source code, it falls back to using a generic pretty printer. So the worst that can happen is that your changes trigger some harmless reformatting of your code.

If you really don't care about preserving the original formatting, you can access the pretty printer directly:
```js
var output = recast.prettyPrint(ast, { tabWidth: 2 }).code;
```
And here's the exact `output`:
```js
var add = function(b, a) {
  return a + b;
}
```
Note that the weird formatting was discarded, yet the behavior and abstract structure of the code remain the same.

Motivation
---

The more code you have, the harder it becomes to make big, sweeping changes quickly and confidently. Even if you trust yourself not to make too many mistakes, and no matter how proficient you are with your text editor, changing tens of thousands of lines of code takes precious, non-refundable time.

Is there a better way? Not always! When a task requires you to alter the semantics of many different pieces of code in subtly different ways, your brain inevitably becomes the bottleneck, and there is little hope of completely automating the process. Your best bet is to plan carefully, buckle down, and get it right the first time. Love it or loathe it, that's the way programming goes sometimes.

What I hope to eliminate are the brain-wasting tasks, the tasks that are bottlenecked by keystrokes, the tasks that can be expressed as operations on the _syntactic structure_ of your code. Specifically, my goal is to make it possible for you to run your code through a parser, manipulate the abstract syntax tree directly, subject only to the constraints of your imagination, and then automatically translate those modifications back into source code, without upsetting the formatting of unmodified code.

And here's the best part: when you're done running a Recast script, if you're not completely satisfied with the results, blow them away with `git reset --hard`, tweak the script, and just run it again. Change your mind as many times as you like. Instead of typing yourself into a nasty case of [RSI](http://en.wikipedia.org/wiki/Repetitive_strain_injury), gaze upon your new wells of free time and ask yourself: what next?
