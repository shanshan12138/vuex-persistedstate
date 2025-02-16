# vuex-persistedstate

Persist and rehydrate your [Vuex](http://vuex.vuejs.org/) state between page reloads.

<hr />

[![Build Status](https://img.shields.io/travis/robinvdvleuten/vuex-persistedstate.svg)](https://travis-ci.org/robinvdvleuten/vuex-persistedstate)
[![NPM version](https://img.shields.io/npm/v/vuex-persistedstate.svg)](https://www.npmjs.com/package/vuex-persistedstate)
[![NPM downloads](https://img.shields.io/npm/dm/vuex-persistedstate.svg)](https://www.npmjs.com/package/vuex-persistedstate)
[![Prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
[![MIT license](https://img.shields.io/github/license/robinvdvleuten/vuex-persistedstate.svg)](https://github.com/robinvdvleuten/vuex-persistedstate/blob/master/LICENSE)

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![Code Of Conduct](https://img.shields.io/badge/code%20of-conduct-ff69b4.svg)](https://github.com/robinvdvleuten/vuex-persistedstate/blob/master/.github/code_of_conduct.md)

## Requirements

- [Vue.js](https://vuejs.org) (v2.0.0+)
- [Vuex](http://vuex.vuejs.org) (v2.0.0+)

## Install

```bash
npm install --save vuex-persistedstate
```

The [UMD](https://github.com/umdjs/umd) build is also available on [unpkg](https://unpkg.com):

```html
<script src="https://unpkg.com/vuex-persistedstate/dist/vuex-persistedstate.umd.js"></script>
```

You can find the library on `window.createPersistedState`.

## Usage

```js
import createPersistedState from 'vuex-persistedstate'

const store = new Vuex.Store({
  // ...
  plugins: [createPersistedState()],
})
```

Check out the example on [CodeSandbox](https://codesandbox.io).

[![Edit vuex-persistedstate](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/80k4m2598)

Or configured to use with [js-cookie](https://github.com/js-cookie/js-cookie).

[![Edit vuex-persistedstate with js-cookie](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/xl356qvvkz)

Or configured to use with [secure-ls](https://github.com/softvar/secure-ls)

[![Edit vuex-persistedstate with secure-ls (encrypted data)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/vuex-persistedstate-with-secure-ls-encrypted-data-7l9wb?fontsize=14)

### Nuxt.js

It is possible to use vuex-persistedstate with Nuxt.js. Due to the order code is loaded in, vuex-persistedstate must be included as a NuxtJS plugin:

```javascript
// nuxt.config.js

...
plugins: [{ src: '~/plugins/localStorage.js', ssr: false }]
...
```

```javascript
// ~/plugins/localStorage.js

import createPersistedState from 'vuex-persistedstate'

export default ({store}) => {
  window.onNuxtReady(() => {
    createPersistedState({
        key: 'yourkey',
        paths: [...]
        ...
    })(store)
  })
}
```

## API

### `createPersistedState([options])`

Creates a new instance of the plugin with the given options. The following options
can be provided to configure the plugin for your specific needs:

- `key <String>`: The key to store the persisted state under. (default: **vuex**)
- `paths <Array>`: An array of any paths to partially persist the state. If no paths are given, the complete state is persisted. Paths must be specified using dot notation. If using modules, include the module name. eg: "auth.user" (default: **[]**)
- `reducer <Function>`: A function that will be called to reduce the state to persist based on the given paths. Defaults to include the values.
- `subscriber <Function>`: A function called to setup mutation subscription. Defaults to `store => handler => store.subscribe(handler)`

- `storage <Object>`: Instead for (or in combination with) `getState` and `setState`. Defaults to localStorage.
- `getState <Function>`: A function that will be called to rehydrate a previously persisted state. Defaults to using `storage`.
- `setState <Function>`: A function that will be called to persist the given state. Defaults to using `storage`.
- `filter <Function>`: A function that will be called to filter any mutations which will trigger `setState` on storage eventually. Defaults to `() => true`
- `arrayMerger <Function>`: A function for merging arrays when rehydrating state. Defaults to `function (store, saved) { return saved }` (saved state replaces supplied state).

## Customize Storage

If it's not ideal to have the state of the Vuex store inside localstorage. One can easily implement the functionality to use [cookies](https://github.com/js-cookie/js-cookie) for that (or any other you can think of);

[![Edit vuex-persistedstate with js-cookie](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/xl356qvvkz?autoresize=1)

```js
import { Store } from 'vuex'
import createPersistedState from 'vuex-persistedstate'
import * as Cookies from 'js-cookie'

const store = new Store({
  // ...
  plugins: [
    createPersistedState({
      storage: {
        getItem: key => Cookies.get(key),
        // Please see https://github.com/js-cookie/js-cookie#json, on how to handle JSON.
        setItem: (key, value) =>
          Cookies.set(key, value, { expires: 3, secure: true }),
        removeItem: key => Cookies.remove(key),
      },
    }),
  ],
})
```

In fact, any object following the Storage protocol (getItem, setItem, removeItem, etc) could be passed:

```js
createPersistedState({ storage: window.sessionStorage })
```

This is especially useful when you are using this plugin in combination with server-side rendering, where one could pass an instance of [dom-storage](https://www.npmjs.com/package/dom-storage).

### 🔐Encrypted Local Storage

If you need to use **Local Storage** (or you want to) but needs to protect the content of the data, you can [encrypt it]('https://github.com/softvar/secure-ls').

[![Edit vuex-persistedstate with secure-ls (encrypted data)](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/vuex-persistedstate-with-secure-ls-encrypted-data-7l9wb?fontsize=14)

```js
import { Store } from 'vuex'
import createPersistedState from 'vuex-persistedstate'
import SecureLS from 'secure-ls'
var ls = new SecureLS({ isCompression: false })

// https://github.com/softvar/secure-ls

const store = new Store({
  // ...
  plugins: [
    createPersistedState({
      storage: {
        getItem: key => ls.get(key),
        setItem: (key, value) => ls.set(key, value),
        removeItem: key => ls.remove(key),
      },
    }),
  ],
})
```

### ⚠️ LocalForage ⚠️

As it maybe seems at first sight, it's not possible to pass a [LocalForage](https://github.com/localForage/localForage) instance as `storage` property. This is due the fact that all getters and setters must be synchronous and [LocalForage's methods](https://github.com/localForage/localForage#callbacks-vs-promises) are asynchronous.

## Credits

- Robin van der Vleuten ([@robinvdvleuten](https://twitter.com/robinvdvleuten))
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.
