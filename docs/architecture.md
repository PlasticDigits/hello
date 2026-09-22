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
When the actor is allowlisted and Definition of Ready is in the issue body,
the parent inserts one SQLite job in the queued state and does not execute
it. A client-signed lab POST does not complete this fixture.

| Keep | Leave to the parent |
| --- | --- |
| Title prefix `lab:` | Signature check, allowlist, replay suppression |
| Body lines Given, When, and Then, and the native org webhook sentence | The queued job, its acknowledgement, and the org hook |
| This map and ADR 0001 | Forgejo’s hook delivery record as native-origin proof |
| | VMs, deploy, spend, custody, policy changes |

Hello [#6](https://git.cl8y.com/code/hello/issues/6) is a separate DoR
fixture. It does not block #7. If that fixture’s ADR is already in the tree
when this one lands, keep it and follow ADR 0001’s filename rule.

Authority for anything beyond this documentation is
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
