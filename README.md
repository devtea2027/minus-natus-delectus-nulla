# @devtea2027/minus-natus-delectus-nulla

Manually Pre-Bundling of Vite

[![NPM version](https://img.shields.io/npm/v/@devtea2027/minus-natus-delectus-nulla.svg)](https://npmjs.org/package/@devtea2027/minus-natus-delectus-nulla)
[![NPM Downloads](https://img.shields.io/npm/dm/@devtea2027/minus-natus-delectus-nulla.svg)](https://npmjs.org/package/@devtea2027/minus-natus-delectus-nulla)
[![awesome-vite](https://awesome.re/badge.svg)](https://github.com/vitejs/awesome-vite)

English | [简体中文](https://github.com/devtea2027/minus-natus-delectus-nulla/blob/main/README.zh-CN.md)

- Compatible Browser, Node.js and Electron
- Custom Vite [Pre-Bundling](https://vitejs.dev/guide/dep-pre-bundling.html) content

## Install

```bash
npm i @devtea2027/minus-natus-delectus-nulla -D
```

## Usage

```ts
import optimizer from '@devtea2027/minus-natus-delectus-nulla'

export default {
  plugins: [
    optimizer({
      vue: `const vue = window.Vue; export { vue as default }`,
    }),
  ]
}
```

#### Load a local file

```ts
optimizer({
  // support nested module id
  // support return Promise
  '@scope/name': () => require('fs/promises').readFile('path', 'utf-8'),
})
```

#### Node.js and Electron

```ts
optimizer({
  // optimize Electron for using `ipcRenderer` in Electron-Renderer
  electron: `const { ipcRenderer } = require('electron'); export { ipcRenderer };`,

  // this means that both 'fs' and 'node:fs' are supported
  // e.g.
  //   `import fs from 'fs'`
  //   or
  //   `import fs from 'node:fs'`
  fs: () => ({
    // this is consistent with the `alias` behavior
    find: /^(node:)?fs$/,
    code: `const fs = require('fs'); export { fs as default };`
  }),
})
```

## Advance

Optimize Node.js ESM packages as CommonJs modules for Node.js/Electron.  
**e.g.** [execa](https://www.npmjs.com/package/execa), [node-fetch](https://www.npmjs.com/package/node-fetch)

You can see 👉 [vite-plugin-esmodule](https://github.com/vite-plugin/vite-plugin-esmodule)

## API <sub><sup>(Define)</sup></sub>

`optimizer(entries[, options])`

```ts
function optimizer(entries: Entries, options?: OptimizerOptions): import('vite').Plugin;
```

```ts
export interface OptimizerArgs {
  /** Generated file cache directory */
  dir: string;
}

export interface ResultDescription {
  /**
   * This is consistent with the `alias` behavior.
   * 
   * e.g.  
   *   `import fs from 'fs'`  
   *   or  
   *   `import fs from 'node:fs'`  
   * 
   * @example
   * {
   *   // This means that both 'fs' and 'node:fs' are supported.
   *   find: /^(node:)?fs$/,
   *   replacement: '/project/node_modules/.@devtea2027/minus-natus-delectus-nulla/fs.js',
   * }
   */
  alias?: {
    find: string | RegExp;
    /**
     * If not explicitly specified, will use the path to the generated file as the default.
     */
    replacement?: string;
  };
  code?: string;
}

export interface Entries {
  [moduleId: string]:
  | string
  | ResultDescription
  | ((args: OptimizerArgs) => string | ResultDescription | Promise<string | ResultDescription | void> | void);
}

export interface OptimizerOptions {
  /**
   * @default ".@devtea2027/minus-natus-delectus-nulla"
   */
  dir?: string;
  resolveId?: ((id: string) => string | Promise<string | void> | void);
}
```

## How to work

Let's use Vue as an example

```js
optimizer({
  vue: `const vue = window.Vue; export { vue as default }`,
})
```

1. Create `node_modules/.@devtea2027/minus-natus-delectus-nulla/vue.js` and contains the following code

```js
const vue = window.Vue; export { vue as default }
```

2. Register a `vue` alias item and add it to `resolve.alias`

```js
{
  resolve: {
    alias: [
      {
        find: 'vue',
        replacement: '/User/work-directory/node_modules/.@devtea2027/minus-natus-delectus-nulla/vue',
      },
    ],
  },
}

/**
 * 🚧
 * If you are using a function and have no return value, alias will not be registered.
 * In this case, you must explicitly specify alias.
 * 
 * e.g.
 * 
 * optimizer({
 *   async vue(args) {
 * 
 *     // ① You can customize the build `vue` and output it to the specified folder.
 *     await require('vite').build({
 *       entry: require.resolve('vue'),
 *       outputDir: args.dir + '/vue',
 *     })
 * 
 *     return {
 *       alias: {
 *         find: 'vue',
 *         // ② Make sure `replacement` points to the `vue` outputDir
 *         replacement: args.dir + '/vue',
 *       }
 *     }
 *   },
 * })
 */
```

3. Add `vue` to the `optimizeDeps.exclude` by default.  

```js
export default {
  optimizeDeps: {
    // 🚧 You can avoid this behavior by `optimizeDeps.include`
    exclude: ['vue'],
  },
}
```
