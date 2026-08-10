# Submitting to the Accent CMS Hub

Anyone may list a plugin, theme or starter template published from their own
public repository. **Automated checks are the gate**: there is no human in the
admission path, so a pull request that passes CI is merged, and a review by a
maintainer is what earns a higher trust tier rather than what admits an entry.

The hub is an index. Your artifact stays in your repository, on your release
page, under your control. Nothing here is hosted or rebuilt by AccentX.

## Before you start

Publish the artifact itself first. One gzip-compressed tar per version, named
`<name>-<version>.tar.gz`, with everything under a single `<name>/` directory,
uploaded somewhere permanent under the **same owner** as the repository you
list -- a GitHub release of that repository is the normal choice.

`accent package` builds the archive and prints the manifest for it.

## What a submission looks like

One pull request, adding or changing only these files:

```
<kind>s/<name>/<version>.json        the version manifest -- new file
<kind>s/index.json                   regenerated, not hand-written
authors/<handle>.json                your author record, once
```

A version manifest:

```json
{
  "contract_version": 2,
  "kind": "plugin",
  "name": "acme-forms",
  "version": "1.2.0",
  "accent_version": ">=0.24.0",
  "api_version": "0.1.0",
  "published_at": "2026-08-07T09:00:00Z",
  "trust": "community",
  "author": "acme",
  "distribution": "registry",
  "artifact": {
    "url": "https://github.com/acme/accent-forms/releases/download/v1.2.0/acme-forms-1.2.0.tar.gz",
    "checksum": "sha256:...",
    "size_bytes": 184320
  },
  "pricing": { "model": "free" },
  "yanked": false
}
```

`api_version` is for plugins only. `schema/manifest.schema.json` is the full,
machine-checkable form, and most editors will use it if you point them at it.

**Do not write `entry_signature`, `index_signature` or `record_signature`.**
Only the registry can produce them, and CI rejects a submission that carries
one. They are stamped when your pull request merges.

### The index is generated

`<kind>s/index.json` is derived from the manifests, and hand-editing it is a
CI failure -- the index decides which version an unpinned install resolves to,
so it must never disagree with the manifests it describes. Regenerate it:

```
hub-registry generate .
```

The generator owns `versions`, `latest_version`, `trust`, `author`, `pricing`
and `updated_at`. It preserves your **catalog fields** -- `description`,
`license`, `repository`, `homepage`, `tags`, `screenshots` -- which describe
the entry rather than any one version, so a first submission writes them into
the index entry by hand once and the generator carries them forward.

### Check it before you open the pull request

```
hub-registry validate . --profile submitted --baseline origin/main
```

The tool is published for Linux with each release:
<https://github.com/AccentCMS/hub-registry/releases>.

## What the automated checks enforce

Schema-level, per document:

- every document matches its schema in `schema/`, including
  `contract_version: 2`
- `kind` matches the directory, `version` equals the filename stem, `name` is
  kebab-case
- no signature field on anything your submission adds
- a commercial entry names `purchase_url` and `seller`

Across documents, which is where most of the contract lives:

- **owner match**: every `artifact.url` is under the same owner as
  `repository`. Reviewing one repository while the bytes ship from another
  account would make review meaningless
- `https` everywhere -- the registry URL and every artifact URL
- the committed index matches what the generator produces from the manifests
- `latest_version` appears in `versions` and is not yanked
- `accent_version` parses as a semver requirement, exactly as the client
  parses it before any download
- `trust: official` requires a repository in the `AccentCMS` org;
  `trust: verified` requires `identity_verified` on your author record. Both
  are claims with mechanical requirements, so both are checked rather than
  taken
- `distribution: external` may not claim `verified` without a maintainer
  having obtained and reviewed a delivered artifact
- a template name may not collide with a starter that ships inside the binary
- **immutability**: a published manifest may not change. Only `yanked` and the
  CI-written signature may move. Ship a new version instead
- no manifest is ever deleted -- yank, never delete
- a commercial entry's author record carries a usable Ed25519
  `license_public_key`

By downloading your artifact:

- `artifact.checksum` matches the bytes actually served, and `size_bytes` is
  the real transfer size
- the archive unpacks within the limits a client enforces: a single `<name>/`
  prefix, no path traversal, no symlinks, and bounds on total size, entry
  count, per-entry size and compression ratio

A submission whose archive a client would refuse therefore fails here, at
review time, rather than at install time on somebody's machine.

If a third-party host is simply unreachable, CI says so distinctly and that is
not a rejection of your submission.

## Trust tiers

| Tier | What it means | How you get it |
|------|---------------|----------------|
| `community` | Listed after automated checks only. Not reviewed | The default outcome of a passing pull request |
| `verified` | A maintainer reviewed it before listing | Granted on request, when there is capacity. Never implied by age or popularity |
| `official` | Authored and published by AccentX | Reserved; requires a repository in the `AccentCMS` org |

The CLI names the tier for anything that is not `official` and warns
explicitly on `community`.

A registry signature is **not** a tier and not an endorsement. Every entry is
signed, including `community`. It asserts one thing: this document was
accepted into this registry and has not been altered since. It says nothing
about the code, which the hub does not host, build or run.

## Updating and retracting

- **A new version** is a new manifest file plus a regenerated index. The old
  manifest stays exactly as it is.
- **Retracting** a version means setting `"yanked": true` on its manifest and
  regenerating the index. A yanked version stays resolvable by exact pin, so
  an existing lockfile keeps working, but it is never chosen as `latest`. CI
  re-stamps the signature, so no signature outlives the document it covers.
- Fixing a mistake in a published manifest is not possible by editing it.
  Publish a new version.

## Delisting

An entry is yanked and then delisted when:

- its artifact URL dies, or its checksum stops matching the bytes served
- its repository is deleted, made private, or is no longer reachable
- it is found to be malicious, or to be typosquatting another entry's name

A scheduled link-health job re-fetches every artifact URL and repository and
flags what has rotted. An index rots from the outside, and a registry that
does not notice is worse than no registry.

Delisting removes a listing. It cannot remove software from anyone's machine,
and it is not a takedown of the author's repository, which the hub never
controlled.

## Licensing your artifact

A commercial artifact still **downloads freely** -- the hub does not serve the
bytes and cannot gate them. It requires a licence to *run*: a vendor-signed
Ed25519 JWT, verified against the `license_public_key` in your author record.
You issue keys yourself with any standard JWT library, and AccentX holds no
credential on your behalf.

Set `pricing.seller` to `vendor` when you sell directly.

## Reporting a problem with a listed entry

Open an issue. For a suspected malicious or typosquatting entry, mark it
clearly in the title; delisting does not wait for the author's response.
