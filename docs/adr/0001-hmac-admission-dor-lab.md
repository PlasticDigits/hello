# ADR 0001: Lab fixture for HMAC admission Definition of Ready

Status: **Proposed** (revision 1, 2026-09-22). Pending independent design review.
This document is not architecture approval, not a merge, and not a deploy.

Issue: [code/hello#6](https://git.cl8y.com/code/hello/issues/6)
Parent (closed): [cl8y-agent-control#1](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/1)
Authority for deploy, spend, custody, and policy expansion:
[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297).
A third design-author claim parks `needs_human`
([cl8y-agent-control#432](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/432)).

## Outcome

Issue #6 remains a stable public lab stimulus for the parent admission contract:

> Given a signed Forgejo `issues` delivery, when the actor is allowlisted and
> Definition of Ready is present, then one SQLite job is queued with executed
> false.

`executed: false` is the parent’s words for “inserted and not run.” The parent
job row uses `status = queued`. The webhook handler does not start a worker
and does not create a VM. There is no `executed` column to add in this
repository.

The observable acknowledgement, already posted on this issue on 2026-09-03,
is the parent’s fixed template: queued gate name, job id, and the clause that
the job was not executed and no VM was created. That historical comment is
evidence the parent path ran once. It is not approval of this design.

## Context

`code/hello` is the public smoke repository for `git.cl8y.com/code`. It has a
README, provenance note, scanner config, and a Woodpecker workflow. It has no
application server, no database, and no webhook secret.

Parent #1 shipped HMAC-SHA256 verification, allowlist checks, Definition of
Ready checks, replay suppression, and a single queued SQLite job on the
control plane. This lab issue’s body is the Given/When/Then text that counts
as Definition of Ready. Its title uses the prefix `lab:` so the fixture stays
eligible for that path.

Hello issue #7 is a sibling fixture. It requires a native organization
webhook delivery. It is a separate stimulus and is not a dependency of #6.

## Non-goals

- Reimplementing HMAC verification, the allowlist, SQLite, or job execution
  in this repository.
- Adding webhook secrets, signing material, or a delivery endpoint here.
- Changing Woodpecker, Coolify, Forgejo project hooks, or runner registration.
- Expanding who may be allowlisted, what gates may run, or what the agent may
  deploy, spend, or custody.
- Renaming this issue to the `lab(` leftover-fixture form, stripping
  Given/When/Then, or closing it as noise.
- Filing a founder card for this ordinary design.
- Treating title or body keywords as permission to change control-plane policy.

## Decision

Record the lab contract in this ADR and keep the running admission path in
the parent control plane. This repository’s only change is documentation that
tells a later implementer how to keep the fixture admit-eligible.

### Component, state, and interface changes

| Surface | Change |
| --- | --- |
| `docs/adr/0001-hmac-admission-dor-lab.md` | This proposal. |
| `docs/architecture.md` | Short map pointing here. |
| README | One link to the architecture map, in the implementation slice after review. |
| Runtime, schema, HTTP, CI, secrets | Unchanged. |

No new interface is exposed. Forgejo continues to deliver `issues` events to
the parent hook. This repo does not gain a route, a table, or a command.

### Affected invariants

Parent ingress rules stay the authority for admission. The fixture must keep
matching them:

- The delivery is a Forgejo `issues` event with a valid SHA-256 signature.
  SHA-1 alone is not enough. Unknown project and bad signature share one
  unauthorized response and create no job.
- The actor is on the parent allowlist. An empty allowlist does not start
  the parent process. A non-allowlisted actor yields a skipped response and
  no job.
- Definition of Ready is the lowercase Given/When/Then triplet in the body,
  or a parent DoR label. Removing that triplet without adding a DoR label
  makes the delivery skip.
- One open job per project, issue, and gate. A replay of the same delivery
  id does not insert a second row. A later duplicate while that row is open
  is a skip.
- The inserted row is queued. Admission does not execute it and does not
  create a VM.
- The title stays `lab:` plus the rest of the current title. The prefix
  `lab(` is a different parent skip (leftover fixture) and would drop this
  issue out of the contract under test.
- Public comments stay on the parent’s fixed template. This repo must not
  echo bodies, signatures, or secrets.

Local invariant for hello: scanners and the existing main-branch workflow
stay as they are. Documentation must not contain credentials.

### Alternatives

| Alternative | Why it loses |
| --- | --- |
| HMAC verifier and SQLite in hello | Second source of truth, secret custody in a public smoke repo, and a policy surface this issue does not authorize. |
| Toy decision function with no secret | Drifts from the parent and adds code the Woodpecker workflow is not set up to test. |
| Retitle to `lab(` and close | Stops the fixture from exercising allowlisted DoR admission. |
| Edit the parent from this worktree | Wrong repository. Parent #1 is already closed. |
| Treat the 2026-09-03 comment as approval | A status comment is not design review and not #297 authority. |

### Complexity

Added: two short documents, then one README sentence. Removed: nothing.
No new dependency, process, table, or workflow step.

### Migration

None. The documents are additive. The issue body and title stay as they are.
No data backfill and no hook cutover.

### Observability

Success for the parent path remains the single fixed acknowledgement on the
issue: queued gate, job id, not executed, no VM. Failure to admit is absence
of a new job, with the parent’s existing unauthorized or skipped responses.
This repository adds no metric, log stream, or status endpoint.

Do not copy job ids, tokens, or live status documents into new commits.

### Failure modes

| Condition | Result |
| --- | --- |
| Bad or missing SHA-256 signature, or unknown project | Unauthorized, no job. Same response body for both. |
| Actor not allowlisted, or Definition of Ready absent | Skipped, no job. |
| Replay of the same delivery id | No second job. |
| Open job already exists for this issue and gate | Duplicate skip. |
| Webhook flood | Parent rate limit. No VM. |
| Title rewritten to `lab(` or Gherkin removed | Fixture no longer matches the contract. Restore title and body. |
| Someone adds a verifier or secret here | Out of scope. Revert. Custody and policy changes belong to #297. |

### Implementation slices

1. **Design publication (this revision).** Add this ADR and
   `docs/architecture.md`. No other files.
2. **Implement after independent review.** Start from the accepted design
   commit. Preserve both design files. Add one sentence to `README.md`
   linking `docs/architecture.md`. Change no other path.

Dependencies: none in `code/hello`. Parent #1 is closed context in another
repository. Hello #7 is not a blocker. Do not wait on hello #12 or #15.

### Tests

This repo has no unit-test harness. Do not add one for a sentence of prose.

- Existing Woodpecker steps (gitleaks, opengrep, trivy) pass on the
  implementation change.
- `README.md` contains a link to `docs/architecture.md`.
- `docs/architecture.md` links to this ADR.
- Issue title still begins with `lab:`.
- Issue body still contains the Given, When, and Then lines.
- Diff contains no secret, no webhook URL with credentials, and no workflow
  or scanner config edits.

### Rollout

Merge the implementation through the repository’s normal trusted pull request
to protected `main`. The change is documentation. Do not register a webhook,
set a deploy target, or enable a new Woodpecker event. The workflow’s deploy
step already exits cleanly when its app id is unset; leave it unset.

### Rollback

Revert the README sentence and, if necessary, these two documents. Admission
behavior is unchanged by that revert because this repository does not perform
admission.

### Integration completion criteria

- An independent review accepts this revision. The author of this ADR does
  not supply that acceptance.
- `main` contains this ADR, the architecture map, and the README link.
- The issue title and Given/When/Then body are unchanged.
- The tree still has no HMAC implementation, no SQLite database, and no new
  secret.
- No deploy, spend, custody change, or allowlist edit was performed.
