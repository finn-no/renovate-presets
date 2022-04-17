# Renovate bot presets

This repo contain a set of [Renovate Bot](https://www.whitesourcesoftware.com/free-developer-tools/renovate/) presets that modules in this organization can depend on.

The purpose of these presets is to be able to optimize the process of keeping dependencies up to date accross all modules in this organization while minimizing the amount of releases of each module in this organization caused by automatically bumping dependencies.

As an example lets say we have a set of modules using Renovate Bot and Semantic Release structured like so:

```sh
|-- @finn-no/application
|   |-- @finn-no/module-a
|   |   |-- 3rd-party-dependency
|   |   |-- @finn-no/module-b
|   |   |   |-- 3rd-party-dependency
```

`@finn-no/module-a` and `@finn-no/module-b` depend on the same `3rd-party-dependency`.

`@finn-no/module-a` is a "top level module" that, mostly common, an application use as a dependency. `@finn-no/module-b` is **not** a public facing module. Rather, it's an "sub level module" which contains code that is only of interest to top modules and it should never be used as a dependency by an application.

If we were to use the default configuration from Renovate Bot the following would happen when we have Semantic Release in our setup and there is a new version of the `3rd-party-dependency`: `@finn-no/module-a` would get a PR with an update of the `3rd-party-dependency` and when merged a new release of `@finn-no/module-a` would be cut. Meanwhile `@finn-no/module-b` would also get a PR with an update of the `3rd-party-dependency` and when merged a new release of `@finn-no/module-b` would also be cut.

This release of `@finn-no/module-b` would again create a new PR in `@finn-no/module-a` and cause a new release of `@finn-no/module-a` when merged.

In other words; Due to both `@finn-no/module-a` and `@finn-no/module-b` depending on `3rd-party-dependency` an update of `3rd-party-dependency` will cause two releases of `@finn-no/module-a` which is not optimal for `@finn-no/application`.

## Module categorisation

To deal with problems such as that described in the introduction above we separate projects into different categories, applications, top level modules and sub modules, where each category has slightly different groupings and dependency strategies.

In short; Sub level modules receive and merge dependencies as they arrive. Top level modules and applications receive and merge dependencies on a less frequent schedule. This more or less removes the problem outlined above.

Development dependencies is received and merged even less frequent than dependencies.

### Applications

Applications are running servers which is deployed to production and serve somewhat of a API or web service. Applications normaly consume top level modules or 3rd party modules.

Applications have their dependencies updated once every weekday.

To use the preset for aplications within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/application"]
}
```

### Top level modules

Top level modules are FINN developed modules which is "publicly facing". These are modules that other external modules and projects or applications will use as a dependency.

Top level modules have their dependencies updated once a week.

To use the preset for top level modules within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/renovate-presets:top-level-module"]
}
```

### Sub modules

Sub modules are **not** publicly facing. These are internal modules which contain code only of interest to top level modules and should never be used by external projects or applications as dependencies.

Sub modules have their dependencies updated as they are published.

To use the preset for sub modules within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/renovate-presets:sub-level-module"]
}
```

## Dev dependencies

Dev dependencies are updated once a month. Both applicatios, top level and sub level modules have the same dev dependency preset.

## Legacy presets

This repo contain a `base.json` and `js.json` renovate preset. These are kept for legacy reasons until further notice and should not be used in new projects.

## License

See license file.
