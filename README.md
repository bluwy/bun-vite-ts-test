> # Update: Bun has [fully supported Vite](https://github.com/oven-sh/bun/issues/250#issuecomment-1646368911) since v0.7 (released 21 July 2023). This repo is no longer needed.

# Bun + Vite + TS (test)

How far can we get with [bun](https://bun.sh) :eyes:

## Status

**6 November 2022**: There has been progress announced in [ViteConf](https://viteconf.org/2022/replay/bun)! Most of the issues below has been resolved, though I haven't thoroughly checked yet. Follow https://github.com/oven-sh/bun/issues/250 for up-to-date progress.

**3 August 2022**: Does not work yet. `child_process` blocks esbuild. If the import is temporarily removed, the CLI exits on execute without messages, and Vite's JavaScript API doesn't run because `http` isn't implemented.

## Results

Bun does not work with Vite yet, not to mention that it has to work with esbuild, rollup, and postcss first, which are major tools that Vite uses under the hood. However this experiment uncovers features needed if it were to work in Vite itself.

Below are a list of issues found, though you can see the finer changes in the [patch made](./patches/vite%2B3.0.0-beta.9.patch).

- `http`, `http2`, `https` not implemented.
- `child_process` not implemented. Blocks esbuild.
- Regex negative lookbehind not supported in JSC. PITA to refactor.
- When `const require = createRequire()` exists, bun treats the file as CJS, transpiling imports to `require()`, but because `const require` is declared late, it gives ` ReferenceError: Cannot access uninitialized variable`. Can be workaround by avoiding `require` and rename as `_require`. https://github.com/oven-sh/bun/issues/453
- Dynamic `import()` doesn't execute file, as if a no-op.
- ESM imports aren't hoisted.
- `process.stdout` not implemented.
- `path.posix` not implemented. Can be polyfilled with `path.posix = path` but works with posix env only.
- `fileUrlToPath` doesn't accept `file://` strings. Can be fixed by wrapping `new URL(<string>)`.
- `createRequire` doesn't work correctly with `file://` strings. URL however works correctly, and can be fixed by wrapping `new URL(<string>)`.
- `process` assignment like `process.env.NODE_ENV = '<string>'` shouldn't be replaced. Can be workaround by wrapping `eval()`.
- `performance` and `perf_hooks` not implemented. Can be polyfilled with `globalThis.performance`.
- `self` not implemented. Can be polyfilled with `globalThis.self = globalThis`.
- `readline` not implemented.

## Notes

- Tried [bun-utilities](https://github.com/xHyroM/bun-utilities) to polyfill `child_process` for esbuild. I'm not sure if it's 100% compatible, if comparing to esbuild's usage. (See [esbuild temp patch](/patches_temp/esbuild%2B0.14.48.patch))
- Since bun 0.1.6, running `dev` gives `SyntaxError: Cannot declare a const variable twice: '__require'` even if `__require` is only declared once.
- Since bun 0.1.6, accessing `require('events').EventEmitter` gives `SyntaxError: Importing binding name 'EventEmitter' is not found.`. Only happened recently, but the issue may have been hidden before.

## Development

```bash
# Install dependencies (patch-package doesn't work with bun install)
$ npm install

# Run Vite with bun
$ npm run dev

# Run Vite with node
$ npm run node-dev
```

The goal is to make sure any changes made to satisfy bun still works in node.

## License

MIT
