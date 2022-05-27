# Renovate bot presets

This repo contain a set of [Renovate Bot](https://www.whitesourcesoftware.com/free-developer-tools/renovate/) presets that modules in this organization can depend on.

The purpose of these presets is to be able to optimize the process of keeping dependencies up to date accross all modules and applications in FINN while minimizing the amount of releases of each module and deployments of applications caused by automatically bumping dependencies.

## Considerations

As an example lets say we have an application dependeing on a set of modules where we are using Renovate Bot to keep dependencies up to date and Semantic Release to automatically publish versions of modules from a CI. The dependency tree look like so:

```sh
|-- application
|   |-- @finn-no/module-a
|   |   |-- 3rd-party-dependency
|   |   |-- @finn-no/module-b
|   |   |   |-- 3rd-party-dependency
```

Here both `@finn-no/module-a` and `@finn-no/module-a` depend on the same `3rd-party-dependency`.

If we were to use the default configuration from Renovate Bot to keep our application, `@finn-no/module-a` and `@finn-no/module-b` up to date, the following will happen when we have Semantic Release in our setup and there is a new version of the `3rd-party-dependency`:

- `@finn-no/module-a` will get a PR with an update of the `3rd-party-dependency` and when merged a new release of `@finn-no/module-a` will be cut.
- The release of `@finn-no/module-a` will create a new PR in the application.
- Meanwhile `@finn-no/module-b` will also get a PR with an update of the `3rd-party-dependency` and when merged a new release of `@finn-no/module-b` will be cut.
- The release of `@finn-no/module-b` will again create a new PR in `@finn-no/module-a` and cause a new release of `@finn-no/module-a` when merged. Which again will create a new PR in the application.

In other words; Due to both `@finn-no/module-a` and `@finn-no/module-b` depending on the same `3rd-party-dependency` an update of `3rd-party-dependency` will cause two releases of `@finn-no/module-a` and two dependency updates in the application.

This is not optimal and just noise.

## Module categorisation

To deal with problems such as that described in the considerations section above we separate projects into different categories, applications, top level modules and sub modules, where each category has slightly different groupings and dependency strategies.

Looking at our dependency tree again:

```sh
|-- application
|   |-- @finn-no/module-a
|   |   |-- 3rd-party-dependency
|   |   |-- @finn-no/module-b
|   |   |   |-- 3rd-party-dependency
```

In short; `@finn-no/module-a` is a "top level module" which, mostly common, an `application` use as a dependency. `@finn-no/module-b` is an "sub level module" which contains code that is only of interest to "top level modules" and which should never be used as a dependency by an application.

## Overall dependency update strategy

In short; Multiple dependencies is grouped in to larger PRs instead of single PRs for each dependency. Then "sub level modules" receive and merge dependencies as they arrive. "Top level modules" receive and merge dependencies on a less frequent schedule. Applications receive and merge dependencies on an even less frequent schedule.

This more or less removes the problem outlined in the considerations section because the following will then happen:

- When there is an update of `3rd-party-dependency`, `@finn-no/module-b` will get an PR immediately and a new release of `@finn-no/module-b` will be cut.
- At a later point `@finn-no/module-a` will get one PR containing an update of both `3rd-party-dependency` and `@finn-no/module-b` and when merged a new release will be cut.
- At a even later point the application will get one PR with an update of `@finn-no/module-a`.

Development dependencies is received and merged even less frequent than dependencies.

### Applications

Applications are running servers which is deployed to production and serve somewhat of a API or web service. Applications normaly consume top level modules or 3rd party modules.

Applications have all their dependencies grouped into one group which is updated once every weekend.

To use the preset for aplications within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/renovate-presets:application"]
}
```

### Top level modules

Top level modules are FINN developed modules which is "publicly facing". These are modules that other external modules and projects or applications will use as a dependency.

Top level modules have two groups of dependencies. There is one grouping for internal FINN modules and one for external (3-rd party) dependencies. Internal dependencies is grouped into one group which is updated once every day between 18:00 and 06:00. External dependencies is grouped into one group which is updated once every weekend.

To use the preset for top level modules within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/renovate-presets:top-level-module"]
}
```

### Sub modules

Sub modules are **not** publicly facing. These are internal modules which contain code only of interest to top level modules and should never be used by external projects or applications as dependencies.

Sub level modules have two groups of dependencies. There is one grouping for internal FINN modules and one for external (3-rd party) dependencies. Internal dependencies is grouped into one group which is updated updated as they are published. External dependencies is grouped into one group which is updated once every weekend.

To use the preset for sub modules within another module, add the following config to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>finn-no/renovate-presets:sub-level-module"]
}
```

## Development dependencies

Development dependencies are updated once a month. Applicatios, top level and sub level modules have the same dev dependency preset. Development dependencies is also configured to auto merge major releases.

## Legacy presets

This repo contain a `base.json` and `js.json` renovate preset. These are kept for legacy reasons until further notice and should not be used in new projects.

## License

See license file.
