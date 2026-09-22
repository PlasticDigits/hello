# Architecture

`code/hello` is a public smoke repository. It stores lab fixtures and the
documents that explain them. It does not run admission, hold webhook secrets,
or store jobs.

## Native Forgejo HMAC delivery (issue #7)

[ADR 0001](adr/0001-native-forgejo-hmac-delivery.md) is the contract for
[issue #7](https://git.cl8y.com/code/hello/issues/7).

Forgejo’s organization hook worker signs an `issues` delivery for org `code`
and POSTs it to the parent control plane
([cl8y-agent-control#1](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/1)).
Org `code` is autoregistered, and the org hook secret matches the control
plane. When the actor is allowlisted and Definition of Ready is in the issue
body, an admit on `opened`, `reopened`, `labeled`, or `synchronize` inserts
one SQLite job in the queued state and does not execute it. A client-signed
lab POST does not complete this fixture.

Proof is one successful org-hook `issues` delivery for this issue together
with the parent outcome of that same delivery. The `implement` comment from
2026-09-03 and the `design_author` comment from 2026-09-22 do not identify
that delivery. Merging this map and the ADR does not queue the job and does
not close #7.

| Keep | Leave to the parent |
| --- | --- |
| Title prefix `lab:` | Signature check, allowlist, replay suppression, duplicate skip, `label_suppressed` |
| Body lines Given, When, and Then, and the native org webhook sentence | The queued job and the invariant 64 acknowledgement for that delivery |
| This map and ADR 0001 | The org `code` hook and its secret; UI install only as the parent’s live leftover |
| | One delivery record paired with that delivery’s parent outcome |
| | VMs, deploy, spend, custody, policy changes |

Hello [#6](https://git.cl8y.com/code/hello/issues/6) is a separate DoR
fixture. It does not block #7. If that fixture’s ADR is already in the tree
when this one lands, keep it and follow ADR 0001’s filename rule.

Authority for anything beyond this documentation is
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
