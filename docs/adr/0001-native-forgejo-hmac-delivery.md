# ADR 0001: Native Forgejo HMAC delivery lab

Status: **Proposed** (revision 1, 2026-09-22). Pending independent design review.
This document is not architecture approval, not a merge, and not a deploy.

Issue: [code/hello#7](https://git.cl8y.com/code/hello/issues/7)
Parent (live leftover): [cl8y-agent-control#1](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/1)
Authority for deploy, spend, custody, and policy expansion:
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
A third design-author claim parks `needs_human`
([cl8y-agent-control#432](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/432)).

## Outcome

Issue #7 stays the public stimulus for one admission path:

> Given a signed Forgejo `issues` delivery from the forge hook worker, when
> the actor is allowlisted and Definition of Ready is present, then one
> SQLite job is queued with executed false.

The delivery that counts is the one Forgejo’s organization hook worker
signs and sends for org `code`. A client that signs a body and POSTs it
itself — unit test, curl, or any other lab POST — does not complete this
fixture, even when the HMAC would verify.

`executed: false` is the parent’s words for “inserted and not run.” The
parent job row uses `status = queued`. The webhook handler’s JSON response
includes `executed: false`. There is no `executed` column to add in this
repository. Admission does not start a worker and does not create a VM.

The observable acknowledgement is the parent’s fixed template: queued gate
name, job id, and the clause that the job was not executed and no VM was
created. The comment already on this issue from 2026-09-03 matches that
template. It shows a queue happened once. It does not identify the sender,
and it is not approval of this design.

## Context

`code/hello` is the public smoke repository for `git.cl8y.com/code`. It has
a README, provenance note, scanner config, and a Woodpecker workflow. It has
no application server, no database, and no webhook secret.

Parent #1 is the live HMAC admission path on the control plane:
`POST /hooks/forgejo` after SHA-256 verification. Org `code` is
autoregistered. The org hook is type `gitea`, active, JSON, and subscribed
to `issues`, `issue_comment`, `pull_request`, and `repository`. Its secret
is the control plane’s org webhook secret. Forgejo’s hook queue worker
performs the delivery. Installing that hook in the Forgejo org UI is an
accepted live leftover when the controller does not POST it itself.

This issue’s body is the Given/When/Then text that counts as Definition of
Ready, plus the sentence that admission has to be a native org webhook.
The title uses the prefix `lab:` so the fixture stays eligible. The prefix
`lab(` is a different parent classification (leftover fixture) and would
drop this issue out of the contract under test.

Hello issue #6 is a sibling stimulus for the same parent admission rules.
Its design, if published, lives on `cac-design-issue-6` and is not a
dependency of this revision. This revision does not copy that branch.

Product admission actions on an `issues` payload are `opened`, `reopened`,
`labeled`, and `open`. The creating delivery is `opened`. Body edits are
not that delivery.

## Non-goals

- Reimplementing HMAC verification, the allowlist, SQLite, or job execution
  in this repository.
- Adding webhook secrets, signing material, a delivery endpoint, or a copy
  of the org hook here.
- Registering, rotating, redelivering, or test-firing the org webhook.
- Teaching the parent to reject clients by User-Agent or any other spoofable
  header. That would be an ingress policy change under #297, in another
  repository.
- Changing Woodpecker, Coolify, Forgejo project hooks, or runner registration.
- Expanding who may be allowlisted, what gates may run, or what the agent may
  deploy, spend, or custody.
- Renaming this issue to the `lab(` leftover-fixture form, stripping
  Given/When/Then, adding a DoR label, or closing it as noise.
- Filing a founder card for this ordinary design.
- Treating title or body keywords as permission to change control-plane policy.
- Pasting hook URLs, delivery logs, or job ids into this repository.

## Decision

Record the native-delivery contract here and keep the running admission path
in the parent control plane. This repository’s change is documentation that
tells a later implementer how to keep the fixture eligible and which proof
counts.

Native origin is Forgejo’s own hook delivery record for the `code` org
webhook: a successful `issues` delivery whose issue is `code/hello` #7.
Correlate that record with one parent acknowledgement on the issue. The
controller authenticates the body with HMAC and does not authenticate that
the TCP client was Forgejo. This design leaves that boundary where it is.
The forge hook worker is the only sender that creates the Forgejo delivery
record. A lab POST does not.

### Component, state, and interface changes

| Surface | Change |
| --- | --- |
| `docs/adr/0001-native-forgejo-hmac-delivery.md` | This proposal. |
| `docs/architecture.md` | Short map pointing here. |
| README | One link to the architecture map, in the implementation slice after review. If a link to `docs/architecture.md` is already present, leave it. |
| Parent `POST /hooks/forgejo` | Unchanged. Still the only admission interface. |
| Runtime, schema, HTTP, CI, secrets in this repo | Unchanged. |

No new interface is exposed. This repo does not gain a route, a table, or a
command. The parent response on admit remains JSON with `status` `queued`,
the job id, the gate name, and `executed` false. The public comment remains
the fixed template.

If `docs/adr/0001-hmac-admission-dor-lab.md` is already on the implementation
base, store this proposal as
`docs/adr/0002-native-forgejo-hmac-delivery.md` and point the architecture
map at that path. Do not overwrite the sibling ADR.

### Affected invariants

Parent ingress rules stay the authority for admission. The fixture must keep
matching them:

- The delivery is a Forgejo `issues` event with a valid SHA-256 signature
  (`X-Hub-Signature-256`, `X-Forgejo-Signature`, or `X-Gitea-Signature`).
  SHA-1 alone is not enough. Unknown project and bad signature share one
  unauthorized response and create no job. Parent invariants 13 and 31.
- The actor and the issue creator are on the parent allowlist. An empty
  allowlist does not start the parent process. A non-allowlisted actor yields
  a skipped response and no job. Parent invariant 15.
- Definition of Ready is the lowercase Given/When/Then triplet in the body,
  or a parent DoR label (`ready`, `dor`, `definition-of-ready`,
  `agent:ready`). Removing that triplet without adding a DoR label makes the
  delivery skip.
- One open job per project, issue, and gate. A replay of the same delivery
  id does not insert a second row. A later duplicate while that row is open
  is a skip. Parent invariant 14.
- The inserted row is queued. The handler does not execute it and does not
  create a VM. The public comment uses the fixed template and does not echo
  bodies, signatures, or secrets. Parent invariant 17.
- The title stays `lab:` plus the rest of the current title. The prefix
  `lab(` marks a leftover fixture and would drop this issue out of the
  contract under test.
- Proof of this fixture is the Forgejo hook delivery record plus that one
  acknowledgement. A lab POST is not proof.

Local invariant for hello: scanners and the existing main-branch workflow
stay as they are. Documentation must not contain credentials, hook URLs, or
live job ids.

### Alternatives

| Alternative | Why it loses |
| --- | --- |
| HMAC verifier and SQLite in hello | Second source of truth, secret custody in a public smoke repo, and a policy surface this issue does not authorize. |
| Parent rejects any client whose User-Agent is not Forgejo | Spoofable, and an ingress policy change in another repository under #297. |
| Lab POST signed with the org secret to close the fixture | The issue forbids that proof. A live POST can also insert a job and reconcile the gate pool, which is spend. |
| Retitle to `lab(` and close | Stops the fixture from exercising native allowlisted DoR admission. |
| Copy the org hook installer into this repo | Puts org-admin and secret custody in the smoke repo. |
| Edit the parent from this worktree | Wrong repository. Parent #1 remains the live leftover for the Coolify path. |
| Treat the 2026-09-03 comment as native proof and as approval | The comment has no sender identity. A status comment is not design review and not #297 authority. |

### Complexity

Added: two short documents, then one README sentence when the link is
absent. Removed: the reading that a signed lab POST would finish this
fixture. No new dependency, process, table, workflow step, or parent branch.

### Migration

None for runtime. The documents are additive. The issue body and title stay
as they are. No data backfill, no hook cutover, and no redelivery.

When the implementation base already contains the sibling ADR named in the
decision table, use the `0002` filename and extend `docs/architecture.md`
with the section for this issue. Keep any existing section for issue #6.

### Observability

Success for the parent path remains the single fixed acknowledgement on the
issue: queued gate, job id, not executed, no VM. The parent log line for
that insert says the job was queued and not executed. Failure to admit is
absence of a new job, with the parent’s existing unauthorized or skipped
responses.

Native origin is visible in Forgejo’s hook delivery history for the org
webhook. This repository adds no metric, log stream, or status endpoint, and
it does not vendor that history. Do not copy job ids, delivery ids, hook
URLs, tokens, or live status documents into new commits.

### Failure modes

| Condition | Result |
| --- | --- |
| Bad or missing SHA-256 signature, or unknown project | Unauthorized, no job. Same response body for both. |
| Actor or creator not allowlisted, or Definition of Ready absent | Skipped, no job. |
| Replay of the same delivery id | No second job. |
| Open job already exists for this issue and gate | Duplicate skip. |
| Webhook flood | Parent rate limit before the delivery id is stored. No VM. |
| Org hook secret disagrees with the control plane after a recreate | Unauthorized. Repair belongs to the parent hook secret procedure, not this repo. |
| Title rewritten to `lab(` or Gherkin removed | Fixture no longer matches the contract. Restore title and body. Do not “fix” it by lab POST. |
| Lab POST offered as completion evidence | Fixture stays open on that point. Revert any document that treats the POST as proof. |
| Someone adds a verifier, secret, or hook URL here | Out of scope. Revert. Custody and policy changes belong to #297. |
| Redelivery or a new DoR label during implement | Can insert another queued job on the live parent. Do not do that in this slice. |

### Implementation slices

1. **Design publication (this revision).** Add this ADR and
   `docs/architecture.md`. No other files. Do not edit the issue. Do not POST
   to the hook.
2. **Implement after independent review.** Start from the accepted design
   commit. Preserve the design files, applying the `0002` rename only when
   the sibling ADR already occupies `0001`. Add one sentence to `README.md`
   linking `docs/architecture.md` when that link is absent. Change no other
   path. Do not redeliver, relabel, or lab-POST.

Dependencies: none in `code/hello`. Parent #1 is live context in another
repository. Hello #6 is not a blocker. Do not wait on hello #12 or #15.

### Tests

This repo has no unit-test harness. Do not add one for a sentence of prose.
Do not add a test that signs a body and POSTs it.

- Existing Woodpecker steps (gitleaks, opengrep, trivy) pass on the
  implementation change.
- `README.md` contains a link to `docs/architecture.md`.
- `docs/architecture.md` links to this ADR (or to the `0002` path when that
  rename applies).
- Issue title still begins with `lab:`.
- Issue body still contains the Given, When, and Then lines and the native
  org webhook sentence.
- Diff contains no secret, no webhook URL, no job id, and no workflow or
  scanner config edits.

### Rollout

Merge the implementation through the repository’s normal trusted pull request
to protected `main`. The change is documentation. Do not register a webhook,
set a deploy target, or enable a new Woodpecker event. The workflow’s deploy
step already exits cleanly when its app id is unset; leave it unset.

### Rollback

Revert the README sentence and, if necessary, these documents. Admission
behavior is unchanged by that revert because this repository does not perform
admission. Do not roll back by deleting the Forgejo org hook.

### Integration completion criteria

- An independent review accepts this revision. The author of this ADR does
  not supply that acceptance.
- `main` contains this ADR (under the filename the decision table selects),
  the architecture map, and the README link.
- The issue title and Given/When/Then body are unchanged, including the
  sentence that admission is a native org webhook.
- The tree still has no HMAC implementation, no SQLite database, no hook
  URL, and no new secret.
- No deploy, spend, custody change, allowlist edit, redelivery, or lab POST
  was performed to produce the design or the implementation.
- No autonomy or HMAC policy file was edited from this work.
