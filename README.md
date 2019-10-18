# renovate-presets

A central place to add and maintain FINN.no related renovate presets (docs.renovatebot.com/config-presets)

## base

This preset extends the default `config:base` preset in renovate, while adding a few other handy presets like `:maintainLockFilesMonthly`.

### Usage

```json
{
  "extends": ["github>finn-no/renovate-presets:base"]
}
```
