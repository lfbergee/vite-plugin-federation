English | [简体中文](./README-zh.md)
# vite-plugin-federation

<p align="center">
  <a href="https://github.com/originjs/vite-plugin-federation/actions/workflows/ci.yml"><img src="https://github.com/originjs/vite-plugin-federation/actions/workflows/ci.yml/badge.svg?branch=main" alt="Build Status"></a>
  <a href="https://www.npmjs.com/package/@originjs/vite-plugin-federation"><img src="https://badgen.net/npm/v/@originjs/vite-plugin-federation" alt="Version"></a>
  <a href="https://nodejs.org/en/about/releases/"><img src="https://img.shields.io/node/v/vite.svg" alt="Node Compatibility"></a>
  <a href="https://www.npmjs.com/package/@originjs/vite-plugin-federation"><img src="https://badgen.net/npm/license/@originjs/vite-plugin-federation" alt="License"></a>
 </p>

A Vite plugin which support Module Federation.
Inspired by Webpack [Module Federation](https://webpack.js.org/concepts/module-federation/) feature.

## Install

Using npm:

```
npm install @originjs/vite-plugin-federation --save-dev
```

## Usage
The main steps in using federation are:
### Step 1: change the configuration
- for a Vite project, in `vite.config.js`:

```js
import { defineConfig } from 'vite'
import federation from "@originjs/vite-plugin-federation";

export default defineConfig({
  plugins: [
    federation({
      name: 'module-name',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/Button.vue',
      },
      remotes:{
          foo: 'remote_foo'
      }
      shared: ['vue']
    })
  ],
})

```

- for a Rollup project, in `rollup.config.js`:

```js
import federation from '@originjs/vite-plugin-federation'


export default {
  input: 'src/index.js',
  output: {
    format: 'esm',
    dir: 'dist'
  },
  plugins: [
    federation({
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/button'
      },
      shared: ['react']
    })
  ]
}
```

### Step 2: asynchronous references

Vue2, for example

```vue
<script>
export default {
  name: 'App',
  components: {
    RemoteButtonScoped: () => import('remote-simple/remote-simple-button-scoped'),
  }
}
</script>
```



### Step 3: Use Remote module components

Vue2, for example

```
<template>
  <div>
  		<RemoteButtonScoped />
  </div>
</template>
```

## Configuration description

### exposes
#### `name: string`
Required as the module name of the remote module.

#### `filename:string`
As the entry file of the remote module, not required, default is `remoteEntry.js`

As the remote module, the list of components exposed to the public, required for the remote module.
```js
exposes: {
// 'externally exposed component name': 'externally exposed component address'
    '. /remote-simple-button': '. /src/components/Button.vue',
        '. /remote-simple-section': '. /src/components/Section.vue'
},
```
### remotes
The remote module entry file referenced as a local module
  ```js
  remotes: {
    // 'remote module name': 'remote module entry file address'
    'remote-simple': 'http://localhost:5011/remoteEntry.js',
},
  ```

### shared
Dependencies shared by local and remote modules. Local modules need to configure the dependencies of all used remote modules; remote modules need to configure the dependencies of externally provided components.
> configuration information
#### `import: boolean`

The default is `true`, whether to add shared to the module, only for the `remote` side, `remote` will reduce some of the packaging time when this configuration is turned on, because there is no need to package some of the `shared`, but once there is no `shared` module available on the `host` side, it will report an error directly, because there is no fallback module available
#### `shareScope: string`

Default is `defualt`, the shared domain name, just keep the `remote` and `host` sides the same
#### `version: string`

Only works on `host` side, the version of the shared module provided is `version` of the `package.json` file in the shared package by default, you need to configure it manually only if you can't get `version` by this method
#### `requiredVersion: string`

Only for the `remote` side, it specifies the required version of the `host shared` used, when the version of the `host` side does not meet the `requiredVersion` requirement, it will use its own `shared` module, provided that it is not configured with `import=false`, which is not enabled by default
## Examples
+ [basic-host-remote](https://github.com/originjs/vite-plugin-federation/tree/main/packages/examples/basic-host-remote)
+ [simple-react](https://github.com/originjs/vite-plugin-federation/tree/main/packages/examples/simple-react)
+ [vue3-demo](https://github.com/originjs/vite-plugin-federation/tree/main/packages/examples/vue3-demo)
+ [vue2-demo](https://github.com/originjs/vite-plugin-federation/tree/main/packages/examples/vue2-demo)


## Construct

vite-plugin-federation dependencies are required during the development and build process, and global installation is recommended.

- rollup
- vite

Part of example requires dependencies, recommended for global installation.

- lerna
- rimraf

Github CI build, not engineering required:

- playwright-chromium



## Development mode

Since Vite is esbuild-based in development mode, we provide separate support for the development mode to leverage Vite’s high-performance development capabilities in the case of remote module deployment.

### FAQ

#### The component of the remote module can not be displayed properly. The console displays“[Vue warn]: Invalid VNode type: Symbol() (symbol)”

The loading of the respective `Vue`by local and remote modules causes the problem of referencing multiple `Vue`.

Solutions：

Keep the local module relatively consistent with the dependent version shared in the remote module.

remote module's `vite.config.ts`

```ts
federation({
  name: 'common-lib',
  filename: 'remoteEntry.js',
  exposes: {
	'./CommonCounter': './src/components/CommonCounter.vue',
	'./CommonHeader': './src/components/CommonHeader.vue'
  },
  shared: {
	vue: {
	  requiredVersion:'^2.0.0' // If you force the version to be too low, delete or modify it to ^ 3.0.0
	}
  }
})
```



####The remote module failed to load the share of the local module, for example`localhost/:1 Uncaught (in promise) TypeError: Failed to fetch dynamically imported module: http://localhost:8080/node_modules/.cacheDir/vue.js?v=4cd35ed0`

Reason: Vite has auto fetch logic for `IP` and Port when starting the service, no full fetch logic has been found in the `Plugin`, and in some cases a fetch failure may occur.

Solutions：

Explicitly declaring IP, Port, `cacheDir` in the local module ensures that our `Plugin` can correctly fetch and pass the dependent addresses.

Local module's `vite.config.ts`

```ts
export default defineConfig({
  server:{
    https: "http",
    host: "192.168.56.1",
    port: 5100,
  },
  cacheDir: "node_modules/.cacheDir",
}
```



## Wiki

[Design framework](https://github.com/originjs/vite-plugin-federation/wiki)