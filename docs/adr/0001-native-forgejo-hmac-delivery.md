# ADR 0001: Native Forgejo HMAC delivery lab

Status: **Proposed** (revision 2, 2026-09-22). Pending independent design review.
This document is not architecture approval, not a merge, and not a deploy.
Revision 2 retargets the parent citations, binds proof to one delivery, and
separates a documentation merge from admission of this issue.

Issue: [code/hello#7](https://git.cl8y.com/code/hello/issues/7)
Parent (live leftover): [cl8y-agent-control#1](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/1)
Parent rules: [INVARIANTS.md](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/src/branch/main/docs/INVARIANTS.md)
and the [org `code` autoregister runbook](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/src/branch/main/docs/runbooks/agentic-workflows.md).
Authority for deploy, spend, custody, and policy expansion:
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
A third design-author claim parks `needs_human`
([cl8y-agent-control#432](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/432)).

## Outcome

Issue #7 stays the public stimulus for one admission path:

> Given a signed Forgejo `issues` delivery from the forge hook worker, when
> the actor is allowlisted and Definition of Ready is present, then one
> SQLite job is queued with executed false.

The delivery that counts is one successful organization-hook `issues`
delivery Forgejo’s hook worker signs and sends for org `code`, whose issue
is `code/hello` #7. The parent outcome that counts is the outcome of **that
same** delivery. A client that signs a body and POSTs it itself — unit test,
curl, or any other lab POST — does not complete this fixture, even when the
HMAC would verify.

`executed: false` is the parent’s words for “inserted and not run.” The
parent job row uses `status = queued`. The webhook handler’s JSON response
includes `executed: false`. There is no `executed` column to add in this
repository. Admission does not start a worker and does not create a VM.

The Forgejo acknowledgement for an insert is parent invariant 64:
`queued_comment_body` names `Gate::as_str()` and the job UUID. Invariant 17
is the Telegram and intake template set.

Two fixed acknowledgements are already on this issue. Neither names a
delivery id or the hook sender:

- 2026-09-03: queued `implement` (not executed; no VM).
- 2026-09-22: queued `design_author` (not executed; no VM).

On 2026-09-22 the issue was also labeled `agent:implement:done`. Workflow
labels move as later gates run. That label is a status marker and does not
identify a sender. “A queue happened once” does not match this record.
Pairing Forgejo hook history with either existing comment does not meet
the Then.

## Context

`code/hello` is the public smoke repository for `git.cl8y.com/code`. It has
a README, provenance note, scanner config, and a Woodpecker workflow. It has
no application server, no database, and no webhook secret.

Parent #1 is the live HMAC admission path on the control plane:
`POST /hooks/forgejo` after SHA-256 verification. Org `code` is
autoregistered (parent invariant 31). The control plane’s org webhook
secret is the secret of that Forgejo org hook; after a recreate the hook
secret has to match. When the controller does not POST the hook, installing
it in the Forgejo `code` org UI is an accepted live leftover (invariant 31
and the org `code` autoregister runbook). This slice does not register or
edit that hook.

This issue’s body is the Given/When/Then text that counts as Definition of
Ready, plus the sentence that admission has to be a native org webhook.
The title uses the prefix `lab:` so the fixture stays eligible. The prefix
`lab(` is a different parent classification (leftover fixture) and would
drop this issue out of the contract under test.

Hello issue #6 is a sibling stimulus for the same parent admission rules.
Its design, if published, lives on `cac-design-issue-6` and is not a
dependency of this revision. This revision does not copy that branch.

Parent invariant 26 admits an `issues` delivery for `opened`, `reopened`,
`labeled`, and `synchronize`. The creating delivery is `opened`. A body
edit sits outside that set: Gherkin already in the body is sticky Definition
of Ready, and it does not by itself start another admit.

## Non-goals

- Reimplementing HMAC verification, the allowlist, SQLite, or job execution
  in this repository.
- Adding webhook secrets, signing material, a delivery endpoint, or a copy
  of the org hook here.
- Registering, editing, rotating, redelivering, or test-firing the org webhook.
- Repairing a duplicate skip or a `label_suppressed` outcome from this
  repository, including by relabel, redelivery, or a lab POST that would
  manufacture a delivery-to-outcome pair.
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

Native origin is one successful Forgejo delivery record for the `code` org
webhook: an `issues` delivery whose issue is `code/hello` #7. The parent
outcome of that same delivery is the other half of the proof. The controller
authenticates the body with HMAC and does not authenticate that the TCP
client was Forgejo. This design leaves that boundary where it is. The forge
hook worker is the sender that creates the Forgejo delivery record. A lab
POST does not.

The 2026-09-03 `implement` comment and the 2026-09-22 `design_author`
comment stay on the issue as history. They are not that paired outcome.

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
the job id, the gate name, and `executed` false. The public comment on an
insert remains `queued_comment_body` (invariant 64).

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
- An empty `ALLOWED_USERS` list refuses parent startup. Parent invariant 15.
- A non-allowlisted actor gets a skipped response and no job. Parent
  invariant 26.
- The issue creator has to be allowlisted as well as the sender. Parent
  invariant 73. Author is the issue creator, and the sender allowlist from
  invariant 26 still applies.
- Definition of Ready is the lowercase Given/When/Then triplet in the body,
  or a parent DoR label (`ready`, `dor`, `definition-of-ready`,
  `agent:ready`). Removing that triplet without adding a DoR label makes the
  delivery skip (invariant 26).
- The `issues` admit actions are `opened`, `reopened`, `labeled`, and
  `synchronize` (invariant 26). The creating delivery is `opened`. Body
  edits are outside that set.
- Replay of the delivery id does not start a second job. The per-project
  rate limit runs before that id is stored. A 429 does not store the
  delivery id, so a retry of that same id can still admit. Parent
  invariant 14.
- One open job per project, issue, and gate. A later delivery while that
  row is open is a duplicate skip and inserts nothing. Parent invariant 26.
- After a terminal product job, `opened`, `reopened`, `labeled`, or
  `synchronize` with the same trigger set and a new delivery id returns
  `label_suppressed` and inserts nothing. Parent invariant 26. That outcome
  leaves this fixture unproved. This repository does not repair it.
- The inserted row is queued. The handler does not execute it and does not
  create a VM. The Forgejo acknowledgement is invariant 64:
  `queued_comment_body` names `Gate::as_str()` and the job UUID, and it
  does not echo bodies, signatures, or secrets. Invariant 17 is the
  Telegram and intake template set.
- The title stays `lab:` plus the rest of the current title. The prefix
  `lab(` marks a leftover fixture and would drop this issue out of the
  contract under test.
- Proof of this fixture is one successful org-hook `issues` delivery for
  `code/hello` #7 together with the parent outcome of that same delivery.
  A lab POST is not that proof. Neither existing acknowledgement is that
  proof.

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
| Treat either existing acknowledgement as native proof and as approval | Neither comment names the sender or a delivery id. A status comment is not design review and not #297 authority. |
| Relabel or redeliver during implement so a fresh id correlates | Manufactures the missing pair, and can insert another live job. Out of this slice. |

### Complexity

Added: two short documents, then one README sentence when the link is
absent. Removed: the reading that a signed lab POST would finish this
fixture, and the reading that a documentation merge is the Then. No new
dependency, process, table, workflow step, or parent branch.

### Migration

None for runtime. The documents are additive. The issue body and title stay
as they are. No data backfill, no hook cutover, and no redelivery.

When the implementation base already contains the sibling ADR named in the
decision table, use the `0002` filename and extend `docs/architecture.md`
with the section for this issue. Keep any existing section for issue #6.

### Observability

The signal for this fixture is one Forgejo org-hook `issues` delivery for
`code/hello` #7 whose parent handling is the outcome of that delivery. On
an insert, that outcome is the invariant 64 acknowledgement (queued gate
name and job UUID, not executed, no VM) and the parent log line that says
the job was queued and not executed. The two acknowledgements already on
the issue are visible history. They are not this correlation.

Failure to insert is absence of a new job, with the parent’s existing
unauthorized, skipped, duplicate-skip, `label_suppressed`, or 429 response.
A 429 leaves the delivery id unstored (invariant 14).

Native origin is visible in Forgejo’s hook delivery history for the org
webhook. This repository adds no metric, log stream, or status endpoint, and
it does not vendor that history. Do not copy job ids, delivery ids, hook
URLs, tokens, or live status documents into new commits.

### Failure modes

| Condition | Result |
| --- | --- |
| Bad or missing SHA-256 signature, or unknown project | Unauthorized, no job. Same response body for both. |
| Actor not allowlisted, or Definition of Ready absent | Skipped, no job (invariant 26). |
| Creator not allowlisted | No auto-admit (invariant 73). |
| Replay of a delivery id that was already stored | No second job (invariant 14). |
| Webhook flood | 429 before the delivery id is stored (invariant 14). The id stays unstored, so a retry of that same id can still admit. No VM. |
| Open job already exists for this project, issue, and gate | Duplicate skip, nothing inserted (invariant 26). Fixture stays unproved. This repository does not close or forget that row. |
| Terminal product job, then a new delivery id with the same trigger set (`opened`, `reopened`, `labeled`, `synchronize`) | `label_suppressed`, nothing inserted (invariant 26). Fixture stays unproved. This repository does not relabel or redeliver to clear it. |
| Org hook secret disagrees with the control plane after a recreate | Unauthorized. Repair belongs to the parent hook secret procedure, not this repo. |
| Title rewritten to `lab(` or Gherkin removed | Fixture no longer matches the contract. Restore title and body. Do not “fix” it by lab POST. |
| Lab POST offered as completion evidence | Fixture stays open on that point. Revert any document that treats the POST as proof. |
| Hook history paired with the 2026-09-03 or 2026-09-22 comment | Then stays unmet. Those comments do not identify the sender. |
| Someone adds a verifier, secret, or hook URL here | Out of scope. Revert. Custody and policy changes belong to #297. |
| Redelivery, relabel, or lab POST during implement | Can insert another queued job on the live parent, or can pretend the missing correlation exists. Do not do that in this slice. |

### Implementation slices

1. **Design publication (this revision).** Add this ADR and
   `docs/architecture.md`. No other files. Do not edit the issue. Do not POST
   to the hook. Do not redeliver or relabel.
2. **Implement after independent review.** Start from the accepted design
   commit. Preserve the design files, applying the `0002` rename only when
   the sibling ADR already occupies `0001`. Add one sentence to `README.md`
   linking `docs/architecture.md` when that link is absent. Change no other
   path. Do not redeliver, relabel, or lab-POST, including as a way to
   manufacture the missing delivery-to-outcome correlation.

Dependencies: none in `code/hello`. Parent #1 is live context in another
repository. Hello #6 is not a blocker. Do not wait on hello #12 or #15.

### Tests

This repo has no unit-test harness. Do not add one for a sentence of prose.
Do not add a test that signs a body and POSTs it.

Documentation slice, after the implementation change:

- Existing Woodpecker steps (gitleaks, opengrep, trivy) pass.
- `README.md` contains a link to `docs/architecture.md`.
- `docs/architecture.md` links to this ADR (or to the `0002` path when that
  rename applies).
- Issue title still begins with `lab:`.
- Issue body still contains the Given, When, and Then lines and the native
  org webhook sentence.
- Diff contains no secret, no webhook URL, no job id, and no workflow or
  scanner config edits.

Those checks are the documentation slice. They do not show that one SQLite
job was queued, and they do not close #7. Proving the Then takes one
successful org-hook `issues` delivery for this issue and the parent outcome
of that same delivery. This repository does not manufacture that pair.

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

Documentation slice, all of the following:

- An independent review accepts this revision. The author of this ADR does
  not supply that acceptance.
- `main` contains this ADR (under the filename the decision table selects),
  the architecture map, and the README link.
- The issue title still begins with `lab:`.
- The issue body still contains the Given, When, and Then lines and the
  sentence that admission is a native org webhook.
- The tree still has no HMAC implementation, no SQLite database, no hook
  URL, and no new secret.
- No deploy, spend, custody change, allowlist edit, redelivery, relabel, or
  lab POST was performed to produce the design or the implementation.
- No autonomy or HMAC policy file was edited from this work.
- The items above do not satisfy “one SQLite job is queued” and do not
  close #7.

The Then stays unmet until one successful Forgejo org-hook `issues`
delivery for `code/hello` #7 is shown together with the parent outcome of
that same delivery. The 2026-09-03 `implement` comment, the 2026-09-22
`design_author` comment, and `agent:implement:done` do not supply that
pair. A duplicate skip (open job for that gate) or `label_suppressed`
(new delivery id, same trigger set, after a terminal product job) inserts
nothing and leaves the fixture unproved. This repository does not repair
either case.
