# Minimal Reproduction + Fix

Rollup's `package.json` includes an `"exports"` key that exposes a few files and folders.

```json
  "exports": {
    ".": {
      "node": {
        "require": "./dist/rollup.js",
        "import": "./dist/es/rollup.js"
      },
      "default": "./dist/es/rollup.browser.js"
    },
    "./dist/": "./dist/"
  }
```

Attempting to import a file from `dist`, e.g. `require('rollup/dist/loadConfigFile')`, causes a warning in Node 16.7.0:

> (node:3205) [DEP0148] DeprecationWarning: Use of deprecated folder mapping "./dist/" in the "exports" field module resolution of the package at /Users/dceddia/Projects/github-repros/rollup-dist-error-repro/node_modules/rollup/package.json.
> Update this package.json to use a subpath pattern like "./dist/*".
> (Use `node --trace-deprecation ...` to show where the warning was created)

And with Node 17.0.1, this turns into an error:

> Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath './dist/loadConfigFile' is not defined by "exports" in /Users/dceddia/Projects/github-repros/rollup-dist-error-repro/node_modules/rollup/package.json

The warning suggests a fix, something like `"./dist*"`, but unfortunately this doesn't actually work. It doesn't fix the error on 17.0.1, and on 16.7.0 it causes an error instead of a warning:

> Error [ERR_INVALID_PACKAGE_TARGET]: Invalid "exports" target "./dist/*" defined for './dist/' in the package config /Users/dceddia/Projects/github-repros/rollup-dist-error-repro/node_modules/rollup/package.json

The fix seems to be to use `"./dist/*": "./dist/*.js"`. See [this change](https://github.com/dceddia/rollup/blob/export-dist/package.json#L140).

Using just a plain wildcard on the end didn't work -- it failed with a `MODULE_NOT_FOUND` error. It needs to end with `*.js`, apparently.
