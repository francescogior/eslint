# Migrating to v7.0.0

ESLint v7.0.0 is a major release of ESLint. We have made a few breaking changes in this release. This guide is intended to walk you through the breaking changes.

The lists below are ordered roughly by the number of users each change is expected to affect, where the first items are expected to affect the most users.

## Table of Content

### Breaking changes for users

- [Node.js 8 is no longer supported](#drop-node-8)
- [Lint the files that match to `overrides[].files` by default](#additional-lint-targets)
- [The base path of `overrides` and `ignorePatterns` is changed if the config file is given by the `--config`/`--ignore-path` options](#base-path-change)
- [The place where ESLint loads plugins from is changed](#plugin-loading-change)
- [Runtime deprecation warnings on `~/.eslintrc.*` config files](#runtime-deprecation-on-personal-config-files)
- [Default ignore patterns are changed](#default-ignore-patterns)
- [Description in directive comments](#description-in-directive-comments)
- [Node.js/CommonJS rules are deprecated](#deprecate-node-rules)
- [Several rules get strict](#rules-strict)
- [`eslint:recommended` is updated](#eslint-recommended)

### Breaking changes for plugin developers

- [Node.js 8 is no longer supported](#drop-node-8)
- [Lint the files that match to `overrides[].files` by default](#additional-lint-targets)
- [The place where ESLint loads plugins from is changed](#plugin-loading-change)
- [`RuleTester` class get strict](#rule-tester-strict)

### Breaking changes for integration developers

- [Node.js 8 is no longer supported](#drop-node-8)
- [The place where ESLint loads plugins from is changed](#plugin-loading-change)
- [`CLIEngine` class has been deprecated](#deprecate-cliengine)

---

## <a name="drop-node-8"></a> Node.js 8 is no longer supported

Node.js 8 has been at EOL since December 2019. Therefore, we have decided to drop support for it in ESLint 7. We now support the following versions of Node.js:

- Node.js 10 (`10.12.0` and above)
- Anything above Node.js 12

**To address:** Make sure you upgrade to at least Node.js `10.12.0` when using ESLint 7. Mind the Node.js version of your editor when you use ESLint 7 via editor integrations. If you are unable to upgrade, we recommend continuing to use ESLint 6 until you are able to upgrade Node.js.

**Related issue(s):** [RFC44](https://github.com/eslint/rfcs/blob/master/designs/2019-drop-node8/README.md), [#12700](https://github.com/eslint/eslint/pull/12700)

## <a name="additional-lint-targets"></a> Lint the files that match to `overrides[].files` by default

Previously, ESLint lints only `*.js` files by default if directories are present such as below.

```
$ eslint src
```

In the situation, ESLint 7 lints the files that match to `overrides[].files` along with `*.js` files. (But it excepts patterns that end with `*` in order to avoid linting unlimitedly.) For example, if the following config file is present,

```yml
# .eslintrc.yml
extends: my-config-js
overrides:
  - files: "*.ts"
    extends: my-config-ts
```

then the `eslint src` command checks both `*.js` and `*.ts` files in the `src` directory.

**To address:** Add `--ext js` option if you are using `overrides` but don't want to lint other files than `*.js`. The `--ext` CLI option overrides this new behavior completely.

If you maintain plugins which check other kinds of files than `.js`, it may be convenient if the `recommended` preset of your plugin has the `overrides` setting in order to check the files by default.

**Related issue(s):** [RFC20](https://github.com/eslint/rfcs/blob/master/designs/2019-additional-lint-targets/README.md), [#12677](https://github.com/eslint/eslint/pull/12677)

## <a name="base-path-change"></a> The base path of `overrides` and `ignorePatterns` is changed if the config file is given by the `--config`/`--ignore-path` options

ESLint resolves the following paths as relative to the directory path of having the _entry_ config file. (the _entry_ means that the settings of shareable configs are resolved as relative to the directory of having the first extender.)

- Configuration files (`.eslintrc.*`)
    - relative paths in the `overrides[].files` setting
    - relative paths in the `overrides[].excludedFiles` setting
    - paths which start with `/` in the `ignorePatterns` setting
- Ignore files (`.eslintignore`)
    - paths which start with `/`

ESLint 7 keeps this behavior in most cases, but only if the config file is given via either `--config path/to/a-config` or `--ignore-path path/to/a-ignore` options then ESLint resolves those paths as relative to the current working directory rather than the file location.

**To address:** Update those paths if you are using the config file via `--config` or `--ignore-path` CLI options.

**Related issue(s):** [RFC37](https://github.com/eslint/rfcs/blob/master/designs/2019-changing-base-path-in-config-files-that-cli-options-specify/README.md), [#12887](https://github.com/eslint/eslint/pull/12887)

## <a name="plugin-loading-change"></a> The place where ESLint loads plugins from is changed

Previously, ESLint resolves all plugins from the current working directory by default.

Since ESLint 7, it resolves the `plugins` setting as relative to the directory of having the _entry_ config file. (the _entry_ means that the `plugins` setting of shareable configs is resolved as relative to the directory of having the first extender.)

This will not change anything in most cases. If a config file in a subdirectory has `plugins` setting, the plugins will be loaded from the subdirectory (or ancestor directories that include the current working directory if not found).

If you are using a config file of a shared location via `--config` option, the plugins the config file declare will be loaded from the shared location.

**To address:** Install plugins to the proper place or add `--resolve-plugins-relative-to .` option to override this change.

**Related issue(s):** [RFC47](https://github.com/eslint/rfcs/blob/master/designs/2019-plugin-loading-improvement/README.md), [#12922](https://github.com/eslint/eslint/pull/12922)

## <a name="runtime-deprecation-on-personal-config-files"></a> Runtime deprecation warnings on `~/.eslintrc.*` config files

Personal config files have been deprecated since [v6.7.0](https://eslint.org/blog/2019/11/eslint-v6.7.0-released). As planned, v7.0.0 has started to print runtime deprecation warnings. It prints the warnings when the following situations:

1. When ESLint loaded `~/.eslintrc.*` config files because project's config was not present.
1. When ESLint ignored `~/.eslintrc.*` config files because project's config was present. (in other words, the HOME directory is an ancestor directory of the project and the project config didn't have `root:true` setting.)

**To address:** Remove `~/.eslintrc.*` config files then add `.eslintrc.*` config files to your project directory. Or use `--config` option to use shared config files.

**Related issue(s):** [RFC32](https://github.com/eslint/rfcs/tree/master/designs/2019-deprecating-personal-config/README.md), [#12678](https://github.com/eslint/eslint/pull/12678)

## <a name="default-ignore-patterns"></a> Default ignore patterns are changed

Previously, ESLint ignores the following files:

- Dotfiles (`.*`)
- `node_modules` in the current working directory (`/node_modules/*`)
- `bower_components` in the current working directory (`/bower_components/*`)

ESLint 7 ignores `node_modules/*` of subdirectories as well, but no longer ignores `bower_components/*` and `.eslintrc.js`. Therefore, the new default ignore patterns are:

- Dotfiles except `.eslintrc.*` (`.*` but not `.eslintrc.*`)
- `node_modules` (`/**/node_modules/*`)

**To address:** Modify your `.eslintignore` or the `ignorePatterns` property of your config file if you don't want to lint `bower_components/*` and `.eslintrc.js`.

**Related issue(s):** [RFC51](https://github.com/eslint/rfcs/blob/master/designs/2019-update-default-ignore-patterns/README.md), [#12888](https://github.com/eslint/eslint/pull/12888)

## <a name="description-in-directive-comments"></a> Description in directive comments

Previously, ESLint doesn't have the official way to write the reason for directive comments like `/*eslint-disable*/`, but it's better if directive comments have the reason description.

ESLint 7 ignores the part preceded by `--` in directive comments. For example:

```js
// eslint-disable-next-line a-rule, another-rule -- those are buggy!!
```

**To address:** If you have `--` surrounded by whitespaces in the configuration in directive comments, consider to move it to your config file.

**Related issue(s):** [RFC33](https://github.com/eslint/rfcs/blob/master/designs/2019-description-in-directive-comments/README.md), [#12699](https://github.com/eslint/eslint/pull/12699)

## <a name="deprecate-node-rules"></a> Node.js/CommonJS rules are deprecated

Ten rules about Node.js/CommonJS are deprecated and moved to the [eslint-plugin-node](https://github.com/mysticatea/eslint-plugin-node) plugin.

**To address:** As [our deprecation policy](https://eslint.org/docs/user-guide/rule-deprecation), you can continue to use the deprecated rules. But the source code of the deprecated rules has been frozen, so we don't fix any bugs of those rules anymore. Therefore, we recommend using the corresponded plugin rules instead.

| Deprecated Rules                                                             | Alternative                                                                                                                     |
| :--------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------ |
| [callback-return](https://eslint.org/docs/rules/callback-return)             | [node/callback-return](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/callback-return.md)             |
| [global-require](https://eslint.org/docs/rules/global-require)               | [node/global-require](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/global-require.md)               |
| [handle-callback-err](https://eslint.org/docs/rules/handle-callback-err)     | [node/handle-callback-err](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/handle-callback-err.md)     |
| [no-mixed-requires](https://eslint.org/docs/rules/no-mixed-requires)         | [node/no-mixed-requires](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-mixed-requires.md)         |
| [no-new-require](https://eslint.org/docs/rules/no-new-require)               | [node/no-new-require](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-new-require.md)               |
| [no-path-concat](https://eslint.org/docs/rules/no-path-concat)               | [node/no-path-concat](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-path-concat.md)               |
| [no-process-env](https://eslint.org/docs/rules/no-process-env)               | [node/no-process-env](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-process-env.md)               |
| [no-process-exit](https://eslint.org/docs/rules/no-process-exit)             | [node/no-process-exit](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-process-exit.md)             |
| [no-restricted-modules](https://eslint.org/docs/rules/no-restricted-modules) | [node/no-restricted-require](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-restricted-require.md) |
| [no-sync](https://eslint.org/docs/rules/no-sync)                             | [node/no-sync](https://github.com/mysticatea/eslint-plugin-node/blob/v11.1.0/docs/rules/no-sync.md)                             |

**Related issue(s):** [#12898](https://github.com/eslint/eslint/pull/12898)

## <a name="rules-strict"></a> Several rules get strict

Several rules now report more errors as enhancement:

- [accessor-pairs](https://eslint.org/docs/rules/accessor-pairs) rule now recognizes class members by default.
- [array-callback-return](https://eslint.org/docs/rules/array-callback-return) rule now recognizes `flatMap` method.
- [computed-property-spacing](https://eslint.org/docs/rules/computed-property-spacing) rule now recognizes class members by default.
- [func-names](https://eslint.org/docs/rules/func-names) rule now recognizes function declarations in default exports.
- [no-extra-parens](https://eslint.org/docs/rules/no-extra-parens) rule now recognizes parentheses in assignment targets.
- [no-dupe-class-members](https://eslint.org/docs/rules/no-dupe-class-members) rule now recognizes computed keys if it's static.
- [no-magic-number](https://eslint.org/docs/rules/no-magic-number) rule now recognizes bigint literals.
- [radix](https://eslint.org/docs/rules/radix) rule now recognizes invalid numbers at the second parameter of `parseInt()`.
- [use-isnan](https://eslint.org/docs/rules/use-isnan) rule now recognizes class members by default.
- [yoda](https://eslint.org/docs/rules/yoda) rule now recognizes bigint literals.

**To address:** Modify your code if new warnings appeared.

**Related issue(s):** [#12490](https://github.com/eslint/eslint/pull/12490), [#12608](https://github.com/eslint/eslint/pull/12608), [#12670](https://github.com/eslint/eslint/pull/12670), [#12701](https://github.com/eslint/eslint/pull/12701), [#12765](https://github.com/eslint/eslint/pull/12765), [#12837](https://github.com/eslint/eslint/pull/12837), [#12913](https://github.com/eslint/eslint/pull/12913), [#12915](https://github.com/eslint/eslint/pull/12915), [#12919](https://github.com/eslint/eslint/pull/12919)

## <a name="eslint-recommended"></a> `eslint:recommended` is updated

Three new rules are enabled by the `eslint:recommended` preset.

- [no-dupe-else-if](https://eslint.org/docs/rules/no-dupe-else-if)
- [no-import-assign](https://eslint.org/docs/rules/no-import-assign)
- [no-setter-return](https://eslint.org/docs/rules/no-setter-return)

**To address:** Modify your code if new warnings appeared.

**Related issue(s):** [#12920](https://github.com/eslint/eslint/pull/12920)

## <a name="rule-tester-strict"></a> `RuleTester` class get strict

The `RuleTester` now recognizes more mistakes:

- It fails test-cases if the rule uses either of non-standard properties `node.start` and `node.end`. Use `node.range` instead.
- It fails test-cases if the rule provides autofix but the test-case doesn't have the `output` property. Add the `output` property to test autofix.
- It fails test-cases if any unknown properties are found in the objects in the `errors` property.

**To address:** Modify your rule or test-case if existing test-cases failed.

**Related issue(s):** [RFC25](https://github.com/eslint/rfcs/blob/master/designs/2019-rule-tester-improvements/README.md), [#12096](https://github.com/eslint/eslint/pull/12096), [#12955](https://github.com/eslint/eslint/pull/12955)

## <a name="deprecate-cliengine"></a> `CLIEngine` class has been deprecated

[`CLIEngine` class](https://eslint.org/docs/developer-guide/nodejs-api#cliengine) has been deprecated and replaced by new [`ESLint` class](https://eslint.org/docs/developer-guide/nodejs-api#eslint-class).

Because `CLIEngine` class provides synchronous API but we have a bundle of features that we cannot implement on the synchronous API such as parallel linting, ES modules, and printing progress. Therefore, the new `ESLint` class provides only asynchronous API and we deprecate `CLIEngine` class.

**To address:** Use `ESLint` class if you are using `CLIEngine` class. The `ESLint` class has similar methods:

| `CLIEngine`                                  | `ESLint`                           |
| :------------------------------------------- | :--------------------------------- |
| `executeOnFiles(patterns)`                   | `lintFiles(patterns)`              |
| `executeOnText(text, filePath, warnIgnored)` | `lintText(text, options)`          |
| `getFormatter(name)`                         | `loadFormatter(name)`              |
| `getConfigForFile(filePath)`                 | `calculateConfigForFile(filePath)` |
| `isPathIgnored(filePath)`                    | `isPathIgnored(filePath)`          |
| `static outputFixes(results)`                | `static outputFixes(results)`      |
| `static getErrorResults(results)`            | `static getErrorResults(results)`  |
| `static getFormatter(name)`                  | (removed ※1)                       |
| `addPlugin(pluginId, definition)`            | the `plugins` constructor option   |
| `getRules()`                                 | (not implemented yet)              |
| `resolveFileGlobPatterns()`                  | (removed ※2)                       |

- ※1 The `engine.getFormatter()` method had returned the object of loaded packages as-is. It had made hard to add new features to formatters for backward compatibility of third-party formatters. So the new `eslint.loadFormatter()` method returns the adapter object that wraps the object of loaded packages. It will ease adding new features as being able to provide default stuff. The adapter object may access to `ESLint` instance to make default stuff (for example, it uses loaded plugin rules to make `rulesMeta`). Therefore, `ESLint` class doesn't have the static version of `loadFormatter()` method.
- ※2 Since ESLint 6, ESLint uses a different logic from the `resolveFileGlobPatterns()` method to iterate files. Therefore, we obsolate it.

**Related issue(s):** [RFC40](https://github.com/eslint/rfcs/blob/master/designs/2019-move-to-async-api/README.md), [#12939](https://github.com/eslint/eslint/pull/12939)