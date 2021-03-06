# 📔 Module Resolution

Parcel (v1.7.0 and above) supports multiple module resolution strategies out of the box so you don't have to deal with endless relative paths `../../`.

Notable terms:

- **project root**: the directory of the entrypoint specified to Parcel, or the shared root (common parent directory) when multiple entrypoints are specified.
- **package root**: the directory of the nearest module root in `node_modules`.

## Absolute Paths

`/foo` will resolve `foo` relative to the **project root**.

## ~ Tilde Paths

`~/foo` will resolve `foo` relative to the nearest **package root** or, if not found, the **project root**.

## Aliasing

Aliases are supported through the `alias` field in `package.json`.

This example aliases `react` to `preact` and some local custom module that is not in `node_modules`.

```json
// package.json
{
  "name": "some-package",
  "devDependencies": {
    "parcel-bundler": "^1.7.0"
  },
  "alias": {
    "react": "preact-compat",
    "react-dom": "preact-compat",
    "local-module": "./custom/modules"
  }
}
```

Avoid using any special characters in your aliases as some may be used by Parcel and others by 3rd party tools or extensions. For example:

- `~` used by Parcel to resolve [tilde paths](#~-tilde-paths).
- `@` used by npm to resolve npm organisations.

We advise being explicit when defining your aliases, so please **specify file extensions**, otherwise Parcel will need to guess. See [JavaScript Named Exports](#JavaScript-Named-Exports) for an example of this.

## Other Conditions

### JavaScript Named Exports

Alias mappings apply to many asset types and does not specifically support mapping of JavaScript named exports. If you wish to map JS named exports you can do this:

```json
// package.json
{
  "name": "some-package",
  "alias": {
    "ipcRenderer": "./electron-ipc.js" // specify file extension
  }
}
```

and re-export the named export within the aliased file:

```js
// electron-ipc.js
module.exports = require('electron').ipcRenderer
```

### Flow with Absolute or Tilde Resolution

Flow will need to know about your use of absolute path or tilde path module resolution. Using the [module.name_mapper](https://flow.org/en/docs/config/options/#toc-module-name-mapper-regex-string) feature of Flow we can

> Specify a regular expression to match against module names, and a replacement pattern

Given a project with this structure:

```
package.json
.flowconfig
src/
  index.html
  index.js
  components/
    apple.js
    banana.js
```

And `src/index.html` as an entrypoint, our **project root** is the `src/` folder.

Therefore, to map this import correctly:

```javascript
// index.js
import Apple from '/components/apple'
```

We need Flow to replace the leading `/` in `'/components/apple'` with `src/`, resulting in `'src/components/apple'`.

The following setting in our `.flowconfig` achieves this replacement:

```
[options]
module.name_mapper='^\/\(.*\)$' -> '<PROJECT_ROOT>/src/\1'
```

Where `<PROJECT_ROOT>` is a Flow specific identifier indicating the location of our `.flowconfig`

NB: `module.name_mapper` can have multiple entries. This allows support for [Absolute](module_resolution.html#absolute-paths) or [Tilde](module_resolution.html#~-tilde-paths) Path Resolution in addition to [local module aliasing](module_resolution.html#aliasing) support.

### TypeScript ~ Resolution

TypeScript will need to know about your use of the `~` module resolution or alias mappings. Please refer to the [TypeScript Module Resolution docs](https://www.typescriptlang.org/docs/handbook/module-resolution.html) for further information.

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "~*": ["./src/*"]
    }
  }
}
```

### Monorepo Resolution

These are the advised usages with monorepos at this time:

Advised usage:

- use relative paths.
- use `/` for a root path if a root is required.

Unadvised usage:

- **avoid** `~` use within monorepos.

If you're a monorepo user and would like to contribute to these recommendations, please provide example repos when opening issues to support the discussion.
