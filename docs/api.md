---
id: api
title: API
---

```js
const prettier = require("prettier");
```

## `prettier.format(source [, options])`

`format` is used to format text using Prettier. [Options](options.md) may be provided to override the defaults.

```js
prettier.format("foo ( );", { semi: false });
// -> "foo()"
```

## `prettier.check(source [, options])`

`check` checks to see if the file has been formatted with Prettier given those options and returns a `Boolean`.
This is similar to the `--list-different` parameter in the CLI and is useful for running Prettier in CI scenarios.

## `prettier.formatWithCursor(source [, options])`

`formatWithCursor` both formats the code, and translates a cursor position from unformatted code to formatted code.
This is useful for editor integrations, to prevent the cursor from moving when code is formatted.

The `cursorOffset` option should be provided, to specify where the cursor is. This option cannot be used with `rangeStart` and `rangeEnd`.

```js
prettier.formatWithCursor(" 1", { cursorOffset: 2 });
// -> { formatted: '1;\n', cursorOffset: 1 }
```

## `prettier.resolveConfig(filePath [, options])`

`resolveConfig` can be used to resolve configuration for a given source file, passing its path as the first argument.
The config search will start at the file path and continue to search up the directory (you can use `process.cwd()` to start
searching from the current directory).
Or you can pass directly the path of the config file as `options.config` if you don't wish to search for it.
A promise is returned which will resolve to:
* An options object, providing a [config file](configuration.md) was found.
* `null`, if no file was found.

The promise will be rejected if there was an error parsing the configuration file.

If `options.useCache` is `false`, all caching will be bypassed.

```js
const text = fs.readFileSync(filePath, "utf8");
prettier.resolveConfig(filePath).then(options => {
  const formatted = prettier.format(text, options);
})
```

Use `prettier.resolveConfig.sync(filePath [, options])` if you'd like to use sync version.

## `prettier.clearConfigCache()`

As you repeatedly call `resolveConfig`, the file system structure will be cached for performance.
This function will clear the cache. Generally this is only needed for editor integrations that
know that the file system has changed since the last format took place.

## `prettier.getSupportInfo([version])`

Returns an object representing the parsers, languages and file types Prettier
supports.

If `version` is provided (e.g. `"1.5.0"`), information for that version will be
returned, otherwise information for the current version will be returned.

The support information looks like this:

```typescript
{
  languages: Array<{
    name: string,
    since: string,
    parsers: string[],
    group?: string,
    tmScope: string,
    aceMode: string,
    codemirrorMode: string,
    codemirrorMimeType: string,
    aliases?: string[],
    extensions: string[],
    filenames?: string[],
    linguistLanguageId: number,
    vscodeLanguageIds: string[],
  }>
}
```

## Custom Parser API

If you need to make modifications to the AST (such as codemods), or you want to provide an alternate parser, you can do so by setting the `parser` option to a function. The function signature of the parser function is:
```js
(text: string, parsers: object, options: object) => AST;
```

Prettier's built-in parsers are exposed as properties on the `parsers` argument.

```js
prettier.format("lodash ( )", {
  parser(text, { babylon }) {
    const ast = babylon(text);
    ast.program.body[0].expression.callee.name = "_";
    return ast;
  }
});
// -> "_();\n"
```

The `--parser` CLI option may be a path to a node.js module exporting a parse function.
