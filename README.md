# Accent CMS Hub

The public registry for [Accent CMS](https://accentcms.dev) plugins, themes
and starter templates. The `accent` CLI reads it directly over
`raw.githubusercontent.com`: it is static, auditable JSON in git, not a
service.

## The hub is an index, not a content store

Artifacts live in **their authors' own repositories**. This registry holds
pointers and assertions about them -- a URL, a checksum, a trust tier, a
licence -- and never the bytes. Nothing here is hosted, mirrored or rebuilt by
AccentX, and a registry signature says only that a document was accepted here
and has not been altered since. It is not a review of the code.

## Layout

```
hub.json                        Registry descriptor (contract version, kinds)
plugins/
  index.json                    Discovery index -- generated, never hand-edited
  <name>/<version>.json         Version manifest
themes/
  index.json
  <name>/<version>.json
templates/
  index.json
  <name>/<version>.json
authors/<handle>.json           Author record, including a vendor's licence key
schema/*.schema.json            The four JSON Schemas
CONTRIBUTING.md                 How to submit, and what the checks enforce
```

`<name>` is kebab-case; `<version>` is an exact semver string. Names are
unique within a kind, so a theme and a plugin may both be called `contact`.

## Submitting

Open a pull request. Automated checks are the gate -- there is no human in the
admission path -- and [CONTRIBUTING.md](CONTRIBUTING.md) says what they
enforce, what each trust tier means, and when an entry is delisted.

## Contract v2

This tree implements **contract version 2**, and it is a clean break from the
v1 `registry.json` that used to sit at the root. That file is gone, so a
pre-v2 `accent` binary now fails with

```
registry returned HTTP 404 Not Found: https://raw.githubusercontent.com/accentcms/plugin-registry/main/registry.json
```

The remedy is to upgrade. The break is deliberate: v1 was plugin-only, listed
loose files rather than a single checksummed archive, and had no signature and
no yank path. It never held an entry, so nothing was installed from it.

The client reads this registry from `hub.registry_url`, which defaults to
`https://raw.githubusercontent.com/AccentCMS/hub-registry/main`.
