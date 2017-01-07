# level-hookdown
Simple levelup hooks implemented with abstract-leveldown.

```
npm install level-hookdown
```

[![npm][npm-image]][npm-url]
[![Build Status](https://travis-ci.org/hypermodules/level-hookdown.svg?branch=master)](https://travis-ci.org/hypermodules/level-hookdown)
[![dependencies Status](https://david-dm.org/hypermodules/level-hookdown/status.svg)](https://david-dm.org/hypermodules/level-hookdown)
[![devDependencies Status](https://david-dm.org/hypermodules/level-hookdown/dev-status.svg)](https://david-dm.org/hypermodules/level-hookdown?type=dev)

![image of a hook](hook.png)

[npm-image]: https://img.shields.io/npm/v/level-hookdown.svg
[npm-url]: https://www.npmjs.com/package/level-hookdown

## Usage

```js
var hook = require('level-hookdown')
var MemDB = require('memdb')  // or 'level' or other levelup factory
var mdb = MemDB()
var db = hook(mdb)

function prehook (operation, cb) {
  console.log('this should run before the db operation')
  console.log(operation)
  cb()
}

function posthook (operation, cb) {
  console.log('this should run after the db operation')
  console.log(operation)
  cb()
}

db.prehooks.push(prehook)
db.posthooks.push(posthook)

db.put('beep', 'boop', function (err) {
  if (err) throw err
  db.del('beep', function (err) {
    if (err) throw err
    db.batch([
      { type: 'put', key: 'father', value: 'gloop' },
      { type: 'put', key: 'another', value: 'heep' }
    ], function (err) {
      if (err) throw err
      console.log('done')
    })
  })
})

```

## API

#### `hookdb = hook(db, [options])`

Returns a levelup instance that executes a series of pre and post hook functions before `put`, `del`, and `batch` method calls.  Does not mutate `db` and returns a wrapping `levelup` instance similar to how [mafintosh/subleveldown](https://github.com/mafintosh/subleveldown) works.  Composes well with [mafintosh/subleveldown](https://github.com/mafintosh/subleveldown) and [dominictarr/level-sublevel](https://github.com/dominictarr/level-sublevel).

The following `options` can be set when wrapping a `level` with `hook`:

```js
{
  type: 'parallel' | 'series' | 'limit',
  limit: 5,
  open: noop() {}
}
```

- The `type` determines if the hook functions are run in series, parallel or parallel-limit.  Default is `parallel`.
- The limit option is passed to `run-parallel-limit` when `type` is set to `limit`.  The default is 5.
- Open is a function to run before the underlying database is opened if not open.

### Hooks

#### `hookdb.prehooks`

An array of hook functions that are run before `put`, `del`, and `batch` method calls to the `hookdb` instance.  If a hook function in the prehook array returns an `err` object to its callback, the originating `put`, `del` or `batch` operation will not be run on the contained `db` that `hookdb` is wrapping.  A prehook veto convetion could be built on top of this behavior via error handling and retry.

#### `hookdb.posthooks`

An array of functions that are run after sucessful `put`, `del`, and `batch` method calls on the wrapped `db` instance inside a `hookdb` instance.

#### `hookFn(operation, callback)`

Hook functions receive an `operation` object that describes the level operation and a `callback` function.

`hookdb.prehooks` and `hookdb.posthooks` are arrays that you can add, rearrange, and delete hook functions from using standard array methods like `hookdb.prehooks.push(fn)` and `hookdb.posthooks.pop(fn)`.

##### `operation`

The `operation` argument can contain an object that looks like `{key: key, value: value, type:'put', opts}`, `{key: key, type:'del', opts}` or `{ type: 'batch', opts, array: [operationArray] }`.  The `opts` object in the level operation object are the options that get passed through to the wrapped level.

##### `options`

The `options` object can contain the standard options object that gets passed to the wrapped `db` levelup instance contained in a `hookdb`.


## See Also

- [dominictarr/level-hooks](https://github.com/dominictarr/level-hooks)
- [dominictarr/level-post](https://github.com/dominictarr/level-post)
- [dominictarr/level-sublevel](https://github.com/dominictarr/level-sublevel)
- [Level/abstract-leveldown](https://github.com/Level/abstract-leveldown)
