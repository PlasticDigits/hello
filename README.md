# hello

Public smoke repository for `git.cl8y.com/code`.

Owned by org **code**. Day-to-day git identity is **PlasticDigits**, not the
Forgejo instance admin. Scanners and a lab Coolify hook exercise protected
`main` behavior on a small tree; this repo is not a product runtime.

## Merge gate (H15)

Changes reach `main` only through a pull request with Woodpecker context
`ci/woodpecker/pr/woodpecker` and a SHA-pinned `Do: merge`. Direct pushes to
`main` are disabled; do not use `force_merge`.

Catch-all `CODEOWNERS` was removed so Forgejo does not plant official review
requests on every diff. Official CODEOWNERS review is **not** the merge gate
for this repository.

Standing protection and tree contracts:
[docs/architecture.md](docs/architecture.md). Decision record:
[docs/adr/0001-remove-catchall-codeowners.md](docs/adr/0001-remove-catchall-codeowners.md).
