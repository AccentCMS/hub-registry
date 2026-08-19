<!-- CONTRIBUTING.md has the full submission rules; this is the checklist
     version of them. Delete the sections that do not apply. -->

## What kind of change is this?

- [ ] New artifact (first listing)
- [ ] New version of an already-listed artifact
- [ ] Retraction (`"yanked": true`, per CONTRIBUTING's delisting runbook)
- [ ] Author record change
- [ ] Registry maintenance (workflows, schema, docs)

## Submission checklist

- [ ] The release exists **first**: `artifact.url` is live, and it is under
      the same owner as `repository`
- [ ] The index was regenerated with `hub-registry generate .` -- the
      committed index is never edited by hand
- [ ] `hub-registry validate . --profile submitted --baseline origin/main`
      passes locally
- [ ] No published manifest is edited. Only `yanked` may move on an existing
      manifest; anything else ships as a new version

## For a retraction

- [ ] `"yanked": true` is set on the affected version(s) and the index is
      regenerated
- [ ] The pull request links the tracking issue that names the reason

<!-- CI re-stamps registry signatures on merge; do not attempt to produce
     one in the pull request. An unreachable third-party host fails the
     validate run distinctly -- that is not a rejection; re-run it. -->
