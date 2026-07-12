# Accent CMS Plugin Registry

This repository is the public plugin registry for
[Accent CMS](https://accentcms.dev). The `accent plugin` commands read it
directly over `raw.githubusercontent.com` -- it is a static, auditable JSON
structure, not a service.

## Layout

```
registry.json                          Index of all available plugins
plugins/<name>/versions/<ver>.json     Per-version manifest
```

### registry.json

```json
{
  "version": "1",
  "plugins": [
    {
      "name": "example-plugin",
      "description": "What the plugin does",
      "author": "Author or organization",
      "license": "MIT",
      "repository": "https://github.com/author/example-plugin",
      "latest_version": "1.0.0",
      "versions": ["1.0.0"]
    }
  ]
}
```

### Version manifests

`plugins/<name>/versions/<version>.json`:

```json
{
  "name": "example-plugin",
  "version": "1.0.0",
  "api_version": "1",
  "checksum": "sha256:<hex of plugin.wasm>",
  "download_url": "https://github.com/author/example-plugin/releases/download/v1.0.0/plugin.wasm",
  "metadata_url": "https://github.com/author/example-plugin/releases/download/v1.0.0/plugin.toml"
}
```

Plugin artifacts (`plugin.wasm`, `plugin.toml`) are hosted as GitHub
Release assets in each plugin's own repository; every download is verified
against the manifest's SHA-256 checksum by the `accent` binary.

## Status

The registry is live and currently lists no plugins. The plugin hub
(hub.accentcms.dev) is the planned discovery front-end. To publish a
plugin here, open a discussion at
https://github.com/AccentCMS/accent/discussions or write to
hello@accentcms.dev.

You can also point `plugins.registry_url` in your `config.yaml` at your
own registry following this layout.
