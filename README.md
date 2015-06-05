# taim [![npm version](https://badge.fury.io/js/taim.svg)](https://www.npmjs.com/package/taim)

> tʌɪm | measure execution time of functions and promises

<img align="right" width="170" height="81" src="https://raw.githubusercontent.com/raine/taim/media/img.png" />

```js
taim(require)('./package.json');
var Promise = taim(require)('bluebird');
taim('promisify', Promise.promisifyAll)(fs);
```

### it measures

- execution time of a function
- time until a Promise is resolved
- time until a callback function is invoked

## install

```sh
npm install taim
```

## usage
 
#### `taim(label?, Function) → Function`

Returns a decorated version of a function that when invoked, measures and
prints the execution time of the function.

If the function returns a Promise, it will instead measure the time until the
promise is resolved.

You can optionally pass a label that will shown in the output.

---

#### `taim(label?, Promise) → Function`

Wraps a Promise (or a thenable) so that when it resolves, duration from
invoking `taim` to the promise resolving is printed to stderr.

---

#### `taim.cb(label?, Function) → Function`

Returns a decorated version of a function that when invoked with a callback
function as the last argument, measures and prints the time until the
callback is executed.

```js
const sleeper = (cb) => {
  setTimeout(() => cb('took a nap, sorry'), 500);
}

taim.cb(sleeper)(excuse =>
  console.log('the excuse was:', excuse)
)
```

<img src="https://raw.githubusercontent.com/raine/taim/media/sleeper.png" width="274" height="63">

---

#### `taim.pipe(Function...) → Function`
#### `taim.compose(Function...) → Function`

Before dispatching to [Ramda's][ramda] [`pipe`][pipe] or
[`compose`][compose], applies `taim` to each function.

---

#### `taim.pipeP(Function...) → Function`
#### `taim.composeP(Function...) → Function`

Before dispatching to [Ramda's][ramda] [`pipeP`][pipeP] or
[`composeP`][composeP], applies `taim` to each function.

## examples

```js
const Promise = require('bluebird');
const request = Promise.promisify(require('request'));
const taim = require('taim');

// :: () → Promise [String]
const readURLs = require('../lib/read-urls');
const reqHead = taim('req', (uri) => request({ method: 'HEAD', uri }));
const checkURLs = (urls) => urls
  .map(function(url) {
    return reqHead(url).spread(function(res) {
      if (res.statusCode !== 200) throw new Error(res.statusCode);
    });
  }, { concurrency: 1 })

const urls = taim('read urls', readURLs());
taim('all', checkURLs(urls));
```

<img src="https://raw.githubusercontent.com/raine/taim/media/check-urls.png" width="338" height="279">

---

```js
const taim = require('taim');
const Promise = require('bluebird');
const {pipeP, prop, concat, map, split, join} = require('ramda');
const request = Promise.promisify(require('request'));

const makeHeader = concat('# ');
const makeTodo   = concat('- [ ] ');

const getList = taim('getList', (url) =>
  request(url).then(prop(1)))

const makeShoppingList = taim.pipeP(
  getList,
  split('\n'),
  map(makeTodo),
  join('\n'),
  concat(makeHeader('my shopping list\n\n'))
);

makeShoppingList('http://j.mp/my-grocery-shopping-list')
  .then(console.log);
```

<img width="322" height="279" src="https://raw.githubusercontent.com/raine/taim/media/shopping.png" />

---

See also [`treis`][treis], a tool to debug and observe functions.

[treis]: https://github.com/raine/treis
[ramda]: http://ramdajs.com
[pipe]: http://ramdajs.com/docs/#pipe
[compose]: http://ramdajs.com/docs/#compose
[pipeP]: http://ramdajs.com/docs/#pipeP
[composeP]: http://ramdajs.com/docs/#composeP
