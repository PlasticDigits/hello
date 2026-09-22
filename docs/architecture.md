# Architecture

`code/hello` is a public smoke repository. It stores lab fixtures and the
documents that explain them. It does not run admission, hold webhook secrets,
or store jobs.

## HMAC admission lab (issue #6)

[ADR 0001](adr/0001-hmac-admission-dor-lab.md) is the contract for
[issue #6](https://git.cl8y.com/code/hello/issues/6).

A signed Forgejo `issues` delivery for an allowlisted actor, with Definition
of Ready in the issue body, causes the parent control plane
([cl8y-agent-control#1](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/1))
to insert one SQLite job in the queued state and not execute it. This
repository only keeps the fixture eligible for that path.

| Keep | Leave to the parent |
| --- | --- |
| Title prefix `lab:` | Signature check, allowlist, replay suppression |
| Body lines Given, When, and Then | The queued job and its acknowledgement |
| This map and ADR 0001 | VMs, deploy, spend, custody, policy changes |

Hello [#7](https://git.cl8y.com/code/hello/issues/7) is a separate native-hook
fixture. It does not block #6.

Authority for anything beyond this documentation is
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
