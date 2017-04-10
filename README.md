# nanomatch [![NPM version](https://img.shields.io/npm/v/nanomatch.svg?style=flat)](https://www.npmjs.com/package/nanomatch) [![NPM monthly downloads](https://img.shields.io/npm/dm/nanomatch.svg?style=flat)](https://npmjs.org/package/nanomatch)  [![NPM total downloads](https://img.shields.io/npm/dt/nanomatch.svg?style=flat)](https://npmjs.org/package/nanomatch) [![Linux Build Status](https://img.shields.io/travis/jonschlinkert/nanomatch.svg?style=flat&label=Travis)](https://travis-ci.org/jonschlinkert/nanomatch) [![Windows Build Status](https://img.shields.io/appveyor/ci/jonschlinkert/nanomatch.svg?style=flat&label=AppVeyor)](https://ci.appveyor.com/project/jonschlinkert/nanomatch)

> Fast, minimal glob matcher for node.js. Similar to micromatch, minimatch and multimatch, but complete Bash 4.3 wildcard support only (no support for exglobs, posix brackets or braces)

<details>
<summary><strong>Table of Contents</strong></summary>

- [What is nanomatch?](#what-is-nanomatch)
- [Getting started](#getting-started)
  * [Installing nanomatch](#installing-nanomatch)
  * [Usage](#usage)
  * [Documentation](#documentation)
- [API](#api)
- [Options](#options)
  * [options.basename](#optionsbasename)
  * [options.bash](#optionsbash)
  * [options.cache](#optionscache)
  * [options.dot](#optionsdot)
  * [options.failglob](#optionsfailglob)
  * [options.ignore](#optionsignore)
  * [options.matchBase](#optionsmatchbase)
  * [options.nocase](#optionsnocase)
  * [options.nodupes](#optionsnodupes)
  * [options.nonegate](#optionsnonegate)
  * [options.nonull](#optionsnonull)
  * [options.nullglob](#optionsnullglob)
  * [options.snapdragon](#optionssnapdragon)
  * [options.unescape](#optionsunescape)
  * [options.unixify](#optionsunixify)
- [Features](#features)
- [Bash expansion libs](#bash-expansion-libs)
- [Benchmarks](#benchmarks)
  * [Running benchmarks](#running-benchmarks)
  * [Latest results](#latest-results)
- [About](#about)
  * [Related projects](#related-projects)
  * [Contributing](#contributing)
  * [Running tests](#running-tests)
  * [Author](#author)
  * [License](#license)

</details>

<details>
<summary><strong>Release history</strong></summary>

## History

### key

Changelog entries are classified using the following labels _(from [keep-a-changelog](https://github.com/olivierlacan/keep-a-changelog)_):

* `added`: for new features
* `changed`: for changes in existing functionality
* `deprecated`: for once-stable features removed in upcoming releases
* `removed`: for deprecated features removed in this release
* `fixed`: for any bug fixes
* `bumped`: updated dependencies, only minor or higher will be listed.

### [1.0.4](https://github.com/jonschlinkert/nanomatch/compare/1.0.3...1.0.4) - 2017-04-06

Housekeeping updates. Adds documentation section about escaping, cleans up utils.

### [1.0.3](https://github.com/jonschlinkert/nanomatch/compare/1.0.1...1.0.3) - 2017-04-06

This release includes fixes for windows path edge cases and other improvements for stricter adherence to bash spec.

**Fixed**

* More windows path edge cases

**Added**

* Support for bash-like quoted strings for escaping sequences of characters, such as `foo/"**"/bar` where `**` should be matched literally and not evaluated as special characters.

### [1.0.1](https://github.com/jonschlinkert/nanomatch/compare/1.0.0...1.0.1) - 2016-12-12

**Added**

* Support for windows path edge cases where backslashes are used in brackets or other unusual combinations.

### [1.0.0](https://github.com/jonschlinkert/nanomatch/compare/0.1.0...1.0.0) - 2016-12-12

Stable release.

### [0.1.0] - 2016-10-08

First release.

</details>

## What is nanomatch?

Nanomatch is a fast and accurate glob matcher with full support for standard Bash glob features, including the following "metacharacters": `*`, `**`, `?` and `[...]`.

**Learn more**

* [Getting started](#getting-started): learn how to install and begin using nanomatch
* [Features](#features): jump to info about supported patterns, and a glob matching reference
* [API documentation](#api): jump to available options and methods
* [Unit tests](test): visit unit tests. there is no better way to learn a code library than spending time the unit tests. Nanomatch has 36,000 unit tests - go become a glob matching ninja!

<details>
<summary><strong>How is this different from [minimatch](https://github.com/isaacs/minimatch)?</strong></summary>

**Speed and accuracy**

Nanomatch uses [snapdragon](https://github.com/jonschlinkert/snapdragon) for parsing and compiling globs, which results in:

* Granular control over the entire conversion process in a way that is easy to understand, reason about, and customize.
* Much greater accuracy than minimatch. In fact, nanomatch passes _all of the spec tests_ from bash, including some that bash still fails. However, since there is no real specification for globs, if you encounter a pattern that yields unexpected match results [after researching previous issues](../../issues), [please let us know](../../issues/new).
* Faster matching, from a combination of optimized glob patterns and (optional) caching.

**Basic globbing only**

Nanomatch supports [basic globbing only](#features), which is limited to `*`, `**`, `?` and regex-like brackets.

If you need support for the other [bash "expansion" types](#bash-expansion-libs) (in addition to the wildcard matching provided by nanomatch), consider using [micromatch](https://github.com/jonschlinkert/micromatch) instead. _(micromatch >=3.0.0  uses the nanomatch parser and compiler for basic glob matching)_

</details>

## Getting started

### Installing nanomatch

**Install with [yarn](https://yarnpkg.com/)**

```sh
$ yarn add nanomatch
```

**Install with [npm](https://npmjs.com)**

```sh
$ npm install nanomatch
```

### Usage

Add nanomatch to your project using node's `require()` system:

```js
var nanomatch = require('nanomatch');

// the main export is a function that takes an array of strings to match
// and a string or array of patterns to use for matching
nanomatch(list, patterns[, options]);
```

**Params**

* `list` **{String|Array}**: List of strings to perform matches against. This is often a list of file paths.
* `patterns` **{String|Array}**: One or more [glob paterns](#features) to use for matching.
* `options` **{Object}**: Any [supported options](#options) may be passed

**Examples**

```js
var nm = require('nanomatch');
console.log(nm(['a', 'b/b', 'c/c/c'], '*'));
//=> ['a']

console.log(nm(['a', 'b/b', 'c/c/c'], '*/*'));
//=> ['b/b']

console.log(nm(['a', 'b/b', 'c/c/c'], '**'));
//=> ['a', 'b/b', 'c/c/c']
```

See the [API documentation](#api) for available methods and [options](https://github.com/einaros/options.js).

### Documentation

<details>
<summary><strong>Escaping</strong></summary>

Backslashes and quotes can be used to escape characters, forcing nanomatch to regard those characters as a literal characters.

**Backslashes**

Use backslashes to escape single characters. For example, the following pattern would match `foo/` folled by a literal `*`, followed by zero or more of any characters besides `/`, followed by `/bar`.

```js
'foo/\**/bar'
```

**Quoted strings**

Use single or double quotes to escape sequences of characters. For example, the following patterns would match `foo/**/bar` exactly:

```js
'foo/"**"/bar'
"foo/'**'/bar"
```

</details>

## API

### [nanomatch](index.js#L31)

The main function takes a list of strings and one or more glob patterns to use for matching.

**Params**

* `list` **{Array}**: A list of strings to match
* `patterns` **{String|Array}**: One or more glob patterns to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Array}**: Returns an array of matches

**Example**

```js
var nm = require('nanomatch');
nm(list, patterns[, options]);

console.log(nm(['a.js', 'a.txt'], ['*.js']));
//=> [ 'a.js' ]
```

<details>
<summary><strong>.match</strong></summary>

### [.match](index.js#L99)

Similar to the main function, but `pattern` must be a string.

**Params**

* `list` **{Array}**: Array of strings to match
* `pattern` **{String}**: Glob pattern to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Array}**: Returns an array of matches

**Example**

```js
var nm = require('nanomatch');
nm.match(list, pattern[, options]);

console.log(nm.match(['a.a', 'a.aa', 'a.b', 'a.c'], '*.a'));
//=> ['a.a', 'a.aa']
```

</details>

<details>
<summary><strong>.isMatch</strong></summary>

### [.isMatch](index.js#L162)

Returns true if the specified `string` matches the given glob `pattern`.

**Params**

* `string` **{String}**: String to match
* `pattern` **{String}**: Glob pattern to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Boolean}**: Returns true if the string matches the glob pattern.

**Example**

```js
var nm = require('nanomatch');
nm.isMatch(string, pattern[, options]);

console.log(nm.isMatch('a.a', '*.a'));
//=> true
console.log(nm.isMatch('a.b', '*.a'));
//=> false
```

</details>

<details>
<summary><strong>.not</strong></summary>

### [.not](index.js#L193)

Returns a list of strings that _DO NOT MATCH_ any of the given `patterns`.

**Params**

* `list` **{Array}**: Array of strings to match.
* `patterns` **{String|Array}**: One or more glob pattern to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Array}**: Returns an array of strings that **do not match** the given patterns.

**Example**

```js
var nm = require('nanomatch');
nm.not(list, patterns[, options]);

console.log(nm.not(['a.a', 'b.b', 'c.c'], '*.a'));
//=> ['b.b', 'c.c']
```

</details>

<details>
<summary><strong>.any</strong></summary>

### [.any](index.js#L227)

Returns true if the given `string` matches any of the given glob `patterns`.

**Params**

* `list` **{String|Array}**: The string or array of strings to test. Returns as soon as the first match is found.
* `patterns` **{String|Array}**: One or more glob patterns to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Boolean}**: Returns true if any patterns match `str`

**Example**

```js
var nm = require('nanomatch');
nm.any(string, patterns[, options]);

console.log(nm.any('a.a', ['b.*', '*.a']));
//=> true
console.log(nm.any('a.a', 'b.*'));
//=> false
```

</details>

<details>
<summary><strong>.contains</strong></summary>

### [.contains](index.js#L268)

Returns true if the given `string` contains the given pattern. Similar to [.isMatch](#isMatch) but the pattern can match any part of the string.

**Params**

* `str` **{String}**: The string to match.
* `patterns` **{String|Array}**: Glob pattern to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Boolean}**: Returns true if the patter matches any part of `str`.

**Example**

```js
var nm = require('nanomatch');
nm.contains(string, pattern[, options]);

console.log(nm.contains('aa/bb/cc', '*b'));
//=> true
console.log(nm.contains('aa/bb/cc', '*d'));
//=> false
```

</details>

<details>
<summary><strong>.matchKeys</strong></summary>

### [.matchKeys](index.js#L323)

Filter the keys of the given object with the given `glob` pattern and `options`. Does not attempt to match nested keys. If you need this feature, use [glob-object](https://github.com/jonschlinkert/glob-object) instead.

**Params**

* `object` **{Object}**: The object with keys to filter.
* `patterns` **{String|Array}**: One or more glob patterns to use for matching.
* `options` **{Object}**: Any [options](#options) to change how matches are performed
* `returns` **{Object}**: Returns an object with only keys that match the given patterns.

**Example**

```js
var nm = require('nanomatch');
nm.matchKeys(object, patterns[, options]);

var obj = { aa: 'a', ab: 'b', ac: 'c' };
console.log(nm.matchKeys(obj, '*b'));
//=> { ab: 'b' }
```

</details>

<details>
<summary><strong>.matcher</strong></summary>

### [.matcher](index.js#L352)

Returns a memoized matcher function from the given glob `pattern` and `options`. The returned function takes a string to match as its only argument and returns true if the string is a match.

**Params**

* `pattern` **{String}**: Glob pattern
* `options` **{Object}**: Any [options](#options) to change how matches are performed.
* `returns` **{Function}**: Returns a matcher function.

**Example**

```js
var nm = require('nanomatch');
nm.matcher(pattern[, options]);

var isMatch = nm.matcher('*.!(*a)');
console.log(isMatch('a.a'));
//=> false
console.log(isMatch('a.b'));
//=> true
```

</details>

<details>
<summary><strong>.makeRe</strong></summary>

### [.makeRe](index.js#L418)

Create a regular expression from the given glob `pattern`.

**Params**

* `pattern` **{String}**: A glob pattern to convert to regex.
* `options` **{Object}**: Any [options](#options) to change how matches are performed.
* `returns` **{RegExp}**: Returns a regex created from the given pattern.

**Example**

```js
var nm = require('nanomatch');
nm.makeRe(pattern[, options]);

console.log(nm.makeRe('*.js'));
//=> /^(?:(\.[\\\/])?(?!\.)(?=.)[^\/]*?\.js)$/
```

</details>

<details>
<summary><strong>.create</strong></summary>

### [.create](index.js#L479)

Parses the given glob `pattern` and returns an object with the compiled `output` and optional source `map`.

**Params**

* `pattern` **{String}**: Glob pattern to parse and compile.
* `options` **{Object}**: Any [options](#options) to change how parsing and compiling is performed.
* `returns` **{Object}**: Returns an object with the parsed AST, compiled string and optional source map.

**Example**

```js
var nm = require('nanomatch');
nm.create(pattern[, options]);

console.log(nm.create('abc/*.js'));
// { options: { source: 'string', sourcemap: true },
//   state: {},
//   compilers:
//    { ... },
//   output: '(\\.[\\\\\\/])?abc\\/(?!\\.)(?=.)[^\\/]*?\\.js',
//   ast:
//    { type: 'root',
//      errors: [],
//      nodes:
//       [ ... ],
//      dot: false,
//      input: 'abc/*.js' },
//   parsingErrors: [],
//   map:
//    { version: 3,
//      sources: [ 'string' ],
//      names: [],
//      mappings: 'AAAA,GAAG,EAAC,kBAAC,EAAC,EAAE',
//      sourcesContent: [ 'abc/*.js' ] },
//   position: { line: 1, column: 28 },
//   content: {},
//   files: {},
//   idx: 6 }
```

</details>

<details>
<summary><strong>.parse</strong></summary>

### [.parse](index.js#L518)

Parse the given `str` with the given `options`.

**Params**

* `str` **{String}**
* `options` **{Object}**
* `returns` **{Object}**: Returns an AST

**Example**

```js
var nm = require('nanomatch');
nm.parse(pattern[, options]);

var ast = nm.parse('a/{b,c}/d');
console.log(ast);
// { type: 'root',
//   errors: [],
//   input: 'a/{b,c}/d',
//   nodes:
//    [ { type: 'bos', val: '' },
//      { type: 'text', val: 'a/' },
//      { type: 'brace',
//        nodes:
//         [ { type: 'brace.open', val: '{' },
//           { type: 'text', val: 'b,c' },
//           { type: 'brace.close', val: '}' } ] },
//      { type: 'text', val: '/d' },
//      { type: 'eos', val: '' } ] }
```

</details>

<details>
<summary><strong>.compile</strong></summary>

### [.compile](index.js#L571)

Compile the given `ast` or string with the given `options`.

**Params**

* `ast` **{Object|String}**
* `options` **{Object}**
* `returns` **{Object}**: Returns an object that has an `output` property with the compiled string.

**Example**

```js
var nm = require('nanomatch');
nm.compile(ast[, options]);

var ast = nm.parse('a/{b,c}/d');
console.log(nm.compile(ast));
// { options: { source: 'string' },
//   state: {},
//   compilers:
//    { eos: [Function],
//      noop: [Function],
//      bos: [Function],
//      brace: [Function],
//      'brace.open': [Function],
//      text: [Function],
//      'brace.close': [Function] },
//   output: [ 'a/(b|c)/d' ],
//   ast:
//    { ... },
//   parsingErrors: [] }
```

</details>

<details>
<summary><strong>.clearCache</strong></summary>

### [.clearCache](index.js#L594)

Clear the regex cache.

**Example**

```js
nm.clearCache();
```

</details>

## Options

<details>
<summary><strong>basename</strong></summary>

### options.basename

Allow glob patterns without slashes to match a file path based on its basename. Same behavior as [minimatch](https://github.com/isaacs/minimatch) option `matchBase`.

Type: `Boolean`

Default: `false`

**Example**

```js
nm(['a/b.js', 'a/c.md'], '*.js');
//=> []

nm(['a/b.js', 'a/c.md'], '*.js', {matchBase: true});
//=> ['a/b.js']
```

</details>

<details>
<summary><strong>bash</strong></summary>

### options.bash

Enabled by default, this option enforces bash-like behavior with stars immediately following a bracket expression. Bash bracket expressions are similar to regex character classes, but unlike regex, a star following a bracket expression **does not repeat the bracketed characters**. Instead, the star is treated the same as an other star.

Type: `Boolean`

Default: `true`

**Example**

```js
var files = ['abc', 'ajz'];
console.log(nm(files, '[a-c]*'));
//=> ['abc', 'ajz']

console.log(nm(files, '[a-c]*', {bash: false}));
```

</details>

<details>
<summary><strong>cache</strong></summary>

### options.cache

Disable regex and function memoization.

Type: `Boolean`

Default: `undefined`

</details>

<details>
<summary><strong>dot</strong></summary>

### options.dot

Match dotfiles. Same behavior as [minimatch](https://github.com/isaacs/minimatch) option `dot`.

Type: `Boolean`

Default: `false`

</details>

<details>
<summary><strong>failglob</strong></summary>

### options.failglob

Similar to the `--failglob` behavior in Bash, throws an error when no matches are found.

Type: `Boolean`

Default: `undefined`

</details>

<details>
<summary><strong>ignore</strong></summary>

### options.ignore

String or array of glob patterns to match files to ignore.

Type: `String|Array`

Default: `undefined`

</details>

<details>
<summary><strong>matchBase</strong></summary>

### options.matchBase

Alias for [options.basename](#options-basename).

</details>

<details>
<summary><strong>nocase</strong></summary>

### options.nocase

Use a case-insensitive regex for matching files. Same behavior as [minimatch](https://github.com/isaacs/minimatch).

Type: `Boolean`

Default: `undefined`

</details>

<details>
<summary><strong>nodupes</strong></summary>

### options.nodupes

Remove duplicate elements from the result array.

Type: `Boolean`

Default: `undefined`

**Example**

Example of using the `unescape` and `nodupes` options together:

```js
nm.match(['a/b/c', 'a/b/c'], 'a/b/c');
//=> ['a/b/c', 'a/b/c']

nm.match(['a/b/c', 'a/b/c'], 'a/b/c', {nodupes: true});
//=> ['abc']
```

</details>

<details>
<summary><strong>nonegate</strong></summary>

### options.nonegate

Disallow negation (`!`) patterns, and treat leading `!` as a literal character to match.

Type: `Boolean`

Default: `undefined`

</details>

<details>
<summary><strong>nonull</strong></summary>

### options.nonull

Alias for [options.nullglob](#options-nullglob).

</details>

<details>
<summary><strong>nullglob</strong></summary>

### options.nullglob

If `true`, when no matches are found the actual (arrayified) glob pattern is returned instead of an empty array. Same behavior as [minimatch](https://github.com/isaacs/minimatch) option `nonull`.

Type: `Boolean`

Default: `undefined`

</details>

<details>
<summary><strong>snapdragon</strong></summary>

### options.snapdragon

Pass your own instance of [snapdragon](https://github.com/jonschlinkert/snapdragon) to customize parsers or compilers.

Type: `Object`

Default: `undefined`

</details>

<details>
<summary><strong>unescape</strong></summary>

### options.unescape

Remove backslashes from returned matches.

Type: `Boolean`

Default: `undefined`

**Example**

In this example we want to match a literal `*`:

```js
nm.match(['abc', 'a\\*c'], 'a\\*c');
//=> ['a\\*c']

nm.match(['abc', 'a\\*c'], 'a\\*c', {unescape: true});
//=> ['a*c']
```

</details>

<details>
<summary><strong>unixify</strong></summary>

### options.unixify

Convert path separators on returned files to posix/unix-style forward slashes.

Type: `Boolean`

Default: `true`

**Example**

```js
nm.match(['a\\b\\c'], 'a/**');
//=> ['a/b/c']

nm.match(['a\\b\\c'], {unixify: false});
//=> ['a\\b\\c']
```

</details>

## Features

Nanomatch has full support for standard Bash glob features, including the following "metacharacters": `*`, `**`, `?` and `[...]`.

Here are some examples of how they work:

| **Pattern** | **Description** | 
| --- | --- |
| `*` | Matches any string except for `/`, leading `.`, or `/.` inside a path |
| `**` | Matches any string including `/`, but not a leading `.` or `/.` inside a path. More than two stars (e.g. `***` is treated the same as one star, and `**` loses its special meaning | when it's not the only thing in a path segment, per Bash specifications) |
| `foo*` | Matches any string beginning with `foo` |
| `*bar*` | Matches any string containing `bar` (beginning, middle or end) |
| `*.min.js` | Matches any string ending with `.min.js` |
| `[abc]*.js` | Matches any string beginning with `a`, `b`, or `c` and ending with `.js` |
| `abc?` | Matches `abcd` or `abcz` but not `abcde` |

The exceptions noted for `*` apply to all patterns that contain a `*`.

**Not supported**

The following extended-globbing features are not supported:

* [brace expansion](https://github.com/jonschlinkert/braces) (e.g. `{a,b,c}`)
* [extglobs](https://github.com/jonschlinkert/extglob) (e.g. `@(a|!(c|d))`)
* [POSIX brackets](https://github.com/jonschlinkert/expand-brackets) (e.g. `[[:alpha:][:digit:]]`)

If you need any of these features consider using [micromatch](https://github.com/jonschlinkert/micromatch) instead.

## Bash expansion libs

Nanomatch is part of a suite of libraries aimed at bringing the power and expressiveness of [Bash's](https://www.gnu.org/software/bash/) matching and expansion capabilities to JavaScript, _and - as you can see by the [benchmarks](#benchmarks) - without sacrificing speed_.

| **Related library** | **Matching Type** | **Example** | **Description** | 
| --- | --- | --- | --- |
| `nanomatch` (you are here) | Wildcards | `*` | [Filename expansion](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html#Filename-Expansion), also referred to as globbing and pathname expansion, allows the use of [wildcards](#features) for matching. |
| [expand-tilde](https://github.com/jonschlinkert/expand-tilde) | Tildes | `~` | [Tilde expansion](https://www.gnu.org/software/bash/manual/html_node/Tilde-Expansion.html#Tilde-Expansion) converts the leading tilde in a file path to the user home directory. |
| [braces](https://github.com/jonschlinkert/braces) | Braces | `{a,b,c}` | [Brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html) |
| [expand-brackets](https://github.com/jonschlinkert/expand-brackets) | Brackets | `[[:alpha:]]` | [POSIX character classes](https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html) (also referred to as POSIX brackets, or POSIX character classes) |
| [extglob](https://github.com/jonschlinkert/extglob) | Parens | `!(a\|b)` | [Extglobs](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html#Pattern-Matching) |
| [micromatch](https://github.com/jonschlinkert/micromatch) | All | all | Micromatch is built on top of the other libraries. |

There are many resources available on the web if you want to dive deeper into how these features work in Bash.

## Benchmarks

### Running benchmarks

Install dev dependencies:

```bash
npm i -d && node benchmark
```

### Latest results

```bash
Benchmarking: (6 of 6)
 · globstar-basic
 · large-list-globstar
 · long-list-globstar
 · negation-basic
 · not-glob-basic
 · star-basic

# benchmark/fixtures/match/globstar-basic.js (182 bytes)
  minimatch x 31,046 ops/sec ±0.56% (87 runs sampled)
  multimatch x 27,787 ops/sec ±1.02% (88 runs sampled)
  nanomatch x 453,686 ops/sec ±1.11% (89 runs sampled)

  fastest is nanomatch

# benchmark/fixtures/match/large-list-globstar.js (485686 bytes)
  minimatch x 25.23 ops/sec ±0.46% (44 runs sampled)
  multimatch x 25.20 ops/sec ±0.97% (43 runs sampled)
  nanomatch x 735 ops/sec ±0.66% (89 runs sampled)

  fastest is nanomatch

# benchmark/fixtures/match/long-list-globstar.js (194085 bytes)
  minimatch x 258 ops/sec ±0.87% (83 runs sampled)
  multimatch x 264 ops/sec ±0.90% (82 runs sampled)
  nanomatch x 1,858 ops/sec ±0.56% (89 runs sampled)

  fastest is nanomatch

# benchmark/fixtures/match/negation-basic.js (132 bytes)
  minimatch x 74,240 ops/sec ±1.22% (88 runs sampled)
  multimatch x 25,360 ops/sec ±1.18% (89 runs sampled)
  nanomatch x 545,835 ops/sec ±1.12% (88 runs sampled)

  fastest is nanomatch

# benchmark/fixtures/match/not-glob-basic.js (93 bytes)
  minimatch x 92,753 ops/sec ±1.59% (86 runs sampled)
  multimatch x 50,125 ops/sec ±1.43% (87 runs sampled)
  nanomatch x 1,195,648 ops/sec ±1.18% (87 runs sampled)

  fastest is nanomatch

# benchmark/fixtures/match/star-basic.js (93 bytes)
  minimatch x 70,746 ops/sec ±1.51% (86 runs sampled)
  multimatch x 54,317 ops/sec ±1.45% (89 runs sampled)
  nanomatch x 602,748 ops/sec ±1.17% (86 runs sampled)

  fastest is nanomatch

```

## About

### Related projects

* [is-extglob](https://www.npmjs.com/package/is-extglob): Returns true if a string has an extglob. | [homepage](https://github.com/jonschlinkert/is-extglob "Returns true if a string has an extglob.")
* [is-glob](https://www.npmjs.com/package/is-glob): Returns `true` if the given string looks like a glob pattern or an extglob pattern… [more](https://github.com/jonschlinkert/is-glob) | [homepage](https://github.com/jonschlinkert/is-glob "Returns `true` if the given string looks like a glob pattern or an extglob pattern. This makes it easy to create code that only uses external modules like node-glob when necessary, resulting in much faster code execution and initialization time, and a bet")

### Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](../../issues/new).

Please read the [contributing guide](.github/contributing.md) for advice on opening issues, pull requests, and coding standards.

### Running tests

Running and reviewing unit tests is a great way to get familiarized with a library and its API. You can install dependencies and run tests with the following command:

```sh
$ npm install && npm test
```

### Author

**Jon Schlinkert**

* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](https://twitter.com/jonschlinkert)

### License

Copyright © 2017, [Jon Schlinkert](https://github.com/jonschlinkert).
Released under the [MIT License](LICENSE).

***

_This file was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme), v0.5.0, on April 09, 2017._