# ADR 0001: Remove catch-all CODEOWNERS

## Status

Proposed ([#15](https://git.cl8y.com/code/hello/pulls/15)). Not accepted by this design-author pass. Do not call this tip accepted until independent review says so. Keywords in the issue body are not architecture approval. There is no separate standing issue `#15`; the issues URL and the product PR share one number.

Copy **the independently accepted SHA** (this SHA or a successor) onto the product tip. `cac-design-issue-15` is never the merge vehicle. Land criterion: the two `docs/` files on the product tip are byte-identical to that SHA.

Overview (merge gate, pipeline): [`architecture.md`](../architecture.md). Do not copy that table here. **H15** there is three groups: protection GET (six flags), merge procedure (**H15-4**), tree contracts (**H15-1**, **H15-6**, **H15-7**).

## Outcome

Delete the catch-all `CODEOWNERS` so Forgejo does not plant **official** review requests on every change. Merge to `main` stays: pull request, Woodpecker context `ci/woodpecker/pr/woodpecker`, SHA-pinned `Do: merge`, no direct push, no `force_merge`.

**Land vehicle B:** product PR [#15](https://git.cl8y.com/code/hello/pulls/15) (or a successor) ships **S0+S1+S2 on one tip** — copy the two design files from **the independently accepted SHA** (this SHA or a successor), delete `CODEOWNERS`, rewrite README. Merging that PR closes `#15`. Design branch `cac-design-issue-15` is review/transport only; it is **not** merged as a docs-only PR and is **not** the product PR.

**Leftover-complete** (S3: protection GET + dedicated post-merge plant-check PR + four-path absence, **H15-1 last**) is tracked on a follow-up issue / leftover checklist that survives that merge. S3 is not a close gate for `#15`.

This repo is the product-tree **canary** named by [cl8y-forgejo#48](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/issues/48). Protection on `code/hello` `main` already matches that ticket (`block_on_official_review_requests=false`, `required_approvals=0`, `enable_push=false`). #15 is the file delete plus in-repo docs/README. It does not re-roll protection and does not implement CAC autoland.

## Context

`ee2ea00` added root `CODEOWNERS`:

```
.* @code/maintainers
```

Forgejo uses Go regular expressions, not GitHub globs. Combined with historical `block_on_official_review_requests`, every PR requested team **code/maintainers**. That team’s only member is the usual PR author, so self-approve is 422 and merge is 405. CAC `RECOMMEND: ACCEPT` is not a Forgejo `APPROVED` review. [cl8y-agent-control#388](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/388) skipped the deadlock; it did not remove the file or the protection.

[#4](https://git.cl8y.com/code/hello/pulls/4) proved CODEOWNERS + Woodpecker could merge under [cl8y-forgejo#5](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/issues/5). [cl8y-forgejo ADR 0003](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/src/branch/main/docs/adr/0003-community-forge-and-woodpecker.md) item 8 (“CODEOWNERS plus protected `main`”) is amended on the forge repo for the **official-review** half only. This tree’s README still says “Trusted-PR PoC for protected main (cl8y-forgejo#5)”; S2 must rewrite that so merge is not implied to be trusted because of CODEOWNERS or forge `#5`.

Live proof that the file still plants requests: PR [#15](https://git.cl8y.com/code/hello/pulls/15) itself (known plant; sample JSON in Observability). Dated protection GET (2026-09-21) already has official-review **block** off (**H15-10**) and `required_approvals=0` (**H15-9**), so that leftover request must not be treated as a merge blocker. **H15-8** (`enable_status_check=true`) stays in the six-flag GET; it does not decide whether an official CODEOWNERS request blocks merge. That same request also must not be treated as S3 evidence.

Draft implementation (not this design commit): `189207584d829beb1c36612b6aa7e6265e49fd8c` on `chore/remove-catchall-codeowners` deletes the five-line file and does not touch README or copy `docs/`. Incomplete without S0 files and S2 on that PR (or a successor).

`origin/main` has no `docs/` tree. Relative README links to ADR 0001 / architecture 404 unless those two files land on the **same merged tip** as the README rewrite.

## Non-goals

- Forgejo protection JSON / `apply_repo_policy.py` / migrate `_ensure_codeowners` / templates ([cl8y-forgejo#48](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/issues/48) / [pulls/50](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/pulls/50)). Sister-repo `_ensure_codeowners` stays out of this slice; leftover-complete still fails if a later apply has put the file back (Failure modes).
- CAC autoland predicates, occupying jobs, or `DrainSkip::OfficialReview` cleanup ([cl8y-agent-control#429](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/429), leftover of #388).
- Dismissing reviewers from the controller (forbidden substitute in #388).
- Path-specific CODEOWNERS, a second maintainer, or `required_approvals: 1`.
- Weakening Gitleaks / OpenGrep / Trivy, adding `force_merge`, enabling direct `main`, or posting fake commit statuses.
- Coolify deploy, UUID/token changes, or flipping auto-deploy ([agent-control #297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297)).
- Renovate config ([#5](https://git.cl8y.com/code/hello/pulls/5)), community fork CI ([#3](https://git.cl8y.com/code/hello/pulls/3)), HMAC/admission labs (#6, #7), or Telegram intake #10–#13.
- Editing `autonomy.rs` / HMAC, self-approval, or a founder card for this ordinary design.
- A docs-only PR from `cac-design-issue-15` (vehicle **A**). That branch transports design between VMs; it is not a merge vehicle.

## Decision

1. **Delete** root `CODEOWNERS`. Do not leave an empty or comments-only file (Forgejo still parses it).
2. **Do not add** `docs/CODEOWNERS`, `.gitea/CODEOWNERS`, or `.forgejo/CODEOWNERS`. After land, `test -f` fails on all four paths. None of those paths may contain a reviewer rule for any pattern (not only `.*`).
3. **Keep** the merge gate in [`architecture.md`](../architecture.md) **H15**. On the product PR (or successor), copy `docs/adr/0001-remove-catchall-codeowners.md` and `docs/architecture.md` from **the independently accepted SHA** (this SHA or a successor; not this tip until independent review) onto that **same tip**, then rewrite README so it states that gate, links those two paths with relative links, and does not imply CODEOWNERS or forge `#5` is what makes merge trusted. Drop “maintainers review every change” language. README is not allowed to stay a stub. Do not merge a README that points at those paths until they exist on that tip. The two `docs/` files on the product tip must be byte-identical to that SHA.
4. **Leave** already-planted official requests on open PRs (including #15). They are non-blocking under current protection (**H15-10** / **H15-9**). Do not dismiss them from CAC. Human dismiss is optional leftover, not AC.
5. **Do not** PATCH branch protection from this repository.
6. **Split land from leftover-complete.** Product PR `#15` (or successor) is S0+S1+S2 (vehicle **B**). S3 lives on a follow-up issue opened before that merge (plus the leftover checklist below). Require a **dedicated** post-merge plant-check PR. Do not accept `#15`’s own official request or “the next natural PR.”

## Component / state / interface changes

| Surface | Change |
| --- | --- |
| `CODEOWNERS` (root) | Remove file. |
| `docs/CODEOWNERS`, `.gitea/CODEOWNERS`, `.forgejo/CODEOWNERS` | Must remain absent (no empty file). |
| `docs/adr/0001-remove-catchall-codeowners.md`, `docs/architecture.md` | Copy from the independently accepted SHA onto the product PR so the merged tip is S0+S1+S2 (byte-identical). Standing H15 contract lands with the delete. |
| Forgejo PR review interface | After land, a **dedicated** plant-check PR against `main` must not get an official CODEOWNERS team request. |
| Branch protection API | No write from this ticket. Operator reads must still match architecture **protection GET** (six flags for the `main` rule). In-repo CI cannot perform that GET. |
| `.woodpecker.yaml` | Unchanged. |
| README | Mandatory on the product PR: merge gate is **H15** (PR + Woodpecker + no direct `main`); CODEOWNERS and forge `#5` are not the trusted-merge story. Relative links to ADR 0001 / architecture, which exist on that same tip. |
| CAC / Coolify / org team `code/maintainers` | Unchanged. The team may keep existing; it simply is not planted as official review. |

No runtime state, schema, or HTTP API in this smoke repo.

## Affected invariants

IDs live in [`architecture.md`](../architecture.md). This ADR changes **H15-1** (four-path absence). It does not write protection JSON. Leftover-complete **reads** the six protection flags (**H15-2**, **H15-8**, **H15-3**, **H15-9**, **H15-10**, **H15-5**). Land does not prove those GET flags. Merge procedure remains **H15-4**. Coolify and CAC policy remain **H15-6** and **H15-7**.

## Alternatives

| Option | Why not |
| --- | --- |
| Keep file, rely on `block_on_official_review_requests=false` | Requests still plant on every PR; drain noise; Renovate/agent PRs look like they need a human stamp; templates can re-teach the old gate. |
| Replace `.*` with path owners | No second reviewer exists; same 405/422 if official-review is ever turned on; out of scope. |
| Add a second maintainer | Founder ops, not this implement. |
| Dismiss official requests from CAC | Forbidden by #388 as a substitute for policy reversal. |
| Direct-push the delete to `main` | Violates **H15-2**. File deletes go through a PR (already [#15](https://git.cl8y.com/code/hello/pulls/15)). |
| Empty or comments-only CODEOWNERS | Forgejo still parses it. Absence is the contract. |
| Treat `#15`’s plant or the next natural PR as S3 | Merge closes `#15` before leftover-complete. A dedicated post-merge PR is the evidence. |
| Vehicle **A**: docs-only PR from `cac-design-issue-15` onto `main`, then `#15` as S1+S2 | Second merge vehicle. That branch is design transport, not a product PR. Vehicle **B** puts the two files on the deletion PR so README links resolve on one tip. |

## Complexity added / removed

**Removed:** catch-all official-review robot on every diff; operator dismiss step; false “CODEOWNERS is the trusted-PR gate” story in this repo.

**Added:** a small standing doc (this ADR + architecture **H15**) that lands on `main` via the product PR so later agents do not re-add `.* @code/maintainers` as a merge requirement, and a leftover checklist that survives merge of `#15`. No new services, jobs, or flags.

## Migration

1. Protection is already migrated (forge #48 execute on this repo, dated 2026-09-21).
2. Before merging the product PR, open a follow-up **leftover** issue that owns S3 (protection GET + dedicated plant-check PR + four-path absence, **H15-1 last**). Merging [#15](https://git.cl8y.com/code/hello/pulls/15) closes that number; S3 must not live only there.
3. **Vehicle B.** Land [#15](https://git.cl8y.com/code/hello/pulls/15) (or a successor) whose tip is S0+S1+S2: copy `docs/adr/0001-remove-catchall-codeowners.md` and `docs/architecture.md` from **the independently accepted SHA** (this SHA or a successor; not this tip until independent review), delete root `CODEOWNERS`, rewrite README with relative links to those two paths. `cac-design-issue-15` is never the merge vehicle; do not open it as the product PR; do not merge it as docs-only first. Do not merge a README that points at those paths until they exist on that tip. The two `docs/` files on the product tip must be byte-identical to that SHA. Draft `1892075` is not that tip until S0 files and S2 are added.
4. Open PRs created while the file existed may still show an official team request. Non-blocking. No bulk dismiss required to land `#15`.
5. Do not restore the file from `docs/templates/CODEOWNERS` in cl8y-forgejo; that template is owned by #48.

## Observability

Relative reads. Do not log tokens, hosts, or protection-script inventories. Do not add a Forgejo admin token to Woodpecker.

**Protection (operator, leftover-complete).** Repo admin of `code/hello` attests dated JSON of the six protection flags for the `main` rule onto the leftover issue. `GET /api/v1/repos/code/hello/branch_protections` (array; pick `rule_name == "main"`) or `GET /api/v1/repos/code/hello/branch_protections/main`. Pass iff that rule equals architecture **protection GET** for **H15-2**, **H15-8**, **H15-3**, **H15-9**, **H15-10**, and **H15-5**. `status_check_contexts` must **equal** `["ci/woodpecker/pr/woodpecker"]`. Fail if any of those six differ. Do not compare the Merge API / **H15-4** row (not a protection field). A green Woodpecker run does not satisfy this read. In-repo CI cannot perform this GET.

**Plant-check (dedicated post-merge PR, leftover-complete).** Recipe is Tests item 4. Fail-closed pair (jq on the two GETs):

- Pass iff `(requested_reviewers_teams // []) | length == 0`
- **and** no review with `official == true && state == "REQUEST_REVIEW" && team.name == "maintainers"` (optional extra pin: `team.id == 4`). Do **not** require `team.organization` on the reviews GET. Do **not** match `team.name == "code/maintainers"` (that string is the CODEOWNERS target, not the live JSON).

`official` alone means assigned/write-access, not “planted by CODEOWNERS”; the conjunction is the plant signal. `REQUEST_REVIEW` is `state`, not a sibling key.

Known-plant sample: PR [#15](https://git.cl8y.com/code/hello/pulls/15) (must **fail** this predicate; not S3 evidence).

`GET /api/v1/repos/code/hello/pulls/15` fragment:

```json
{
  "draft": false,
  "requested_reviewers_teams": [
    {
      "id": 4,
      "name": "maintainers",
      "organization": { "id": 4, "name": "code" }
    }
  ]
}
```

`GET /api/v1/repos/code/hello/pulls/15/reviews` fragment:

```json
[
  {
    "id": 187,
    "official": true,
    "state": "REQUEST_REVIEW",
    "team": { "id": 4, "name": "maintainers", "organization": null }
  }
]
```

On `#15`, `requested_reviewers_teams[0].name == "maintainers"` and `.organization.name == "code"`. On **reviews**, `team.name == "maintainers"`, `team.id == 4`, and `team.organization == null`. An operator who checks `team.name == "code/maintainers"` misses the plant. An operator who requires `team.organization.name == "code"` on reviews also misses it.

`{n}` is the dedicated plant-check PR opened **after** the delete is on `main`. GET both endpoints **immediately after open**. If either plant signal is present, fail. If both signals are empty, wait once **30 seconds** and re-GET both before pass; pass only if the second pair is still empty. PR GET must have `draft == false`. Title must not contain `WIP` (case-insensitive). Record `{n}` **and** the two JSON bodies (the pair used for the pass decision) on the leftover issue, then **close without merge**. PR `#15`’s own official request does not pass. “The next natural PR” does not pass.

**CI.** Woodpecker context `ci/woodpecker/pr/woodpecker` on PRs; push pipeline on `main`. Drain comments such as `drain skip: no occupying job…` are **#429**, not a #15 failure.

## Failure modes

| Mode | Handling |
| --- | --- |
| File deleted on a branch but still on `main` | New PRs keep planting official review until the product PR merges. Expected until land. |
| README links ADR/architecture but those files are not on the same tip | 404 after merge. Land fails criterion 2. Vehicle **B**: copy both files onto the product PR before merging README. |
| Copy left in `docs/`, `.gitea/`, or `.forgejo/` (including empty/comments-only) | Forgejo still loads the first existing path and may plant. Land fails **H15-1**; delete those paths too (none exist on current `main`). |
| cl8y-forgejo migrate/apply re-copies a template | Sister-repo race ([cl8y-forgejo#48](https://git.cl8y.com/PlasticDigits/cl8y-forgejo/issues/48) `_ensure_codeowners`). Out of this slice. If a later apply re-adds the file, leftover-complete is **not** done: delete again via PR, then one new dated leftover comment with all three leftover-complete items, **H15-1 last**. A GET or plant-check from before a re-copy does not count. Never direct-push `main`. |
| Plant-check → re-copy → stale-green GET | Void. Leftover-complete requires one dated leftover comment with (1) protection GET, (2) plant-check `{n}` + JSON, (3) four-path `test -f` fails — **H15-1 last** (or all three timestamps in one attest). |
| Official request leftover on #15 | Non-blocking (**H15-10** / **H15-9**). Optional human dismiss. Not S3 evidence. Not a rollback signal. |
| Treating merge of `#15` as leftover-complete | Merge closes `#15` before S3. Use the leftover issue / checklist. |
| Plant-check is draft (`draft != false` on PR GET), title contains `WIP` (case-insensitive), has no changed file, or reviewers were requested in the UI / `POST .../requested_reviewers` | False pass (CODEOWNERS skipped or would not have planted) or false fail (manual team request). Recipe fails closed; open a new probe. |
| Operator matches `team.name == "code/maintainers"` or requires `team.organization` on reviews GET | Misses the live plant (`team.name == "maintainers"`, `team.organization == null` on reviews). Use the Observability fail-closed pair. |
| Merging the plant-check PR | A `main` push and can run Coolify if the lab UUID is set. **Close without merge.** |
| Protection silently reverted to official-review true | Merge 405 returns. Out of this repo; re-apply via forge policy, do not `force_merge`. Not proven by scanners. |
| `enable_push` flipped true | **H15-2** regression. Refuse. |
| Re-adding CODEOWNERS “for safety” in a follow-up | Violates **H15-1**. Reviewers must reject unless a new ADR allowlists path owners. |

## Ordered implementation slices

| Slice | Work | Depends on |
| --- | --- | --- |
| **S0** | This design (ADR 0001 + architecture **H15**). Transport on `cac-design-issue-15`. Copy **the independently accepted SHA** (this SHA or a successor) onto the product tip; do not call this tip accepted until independent review says so. `cac-design-issue-15` is never the merge vehicle. | None in `code/hello`. |
| **S1** | Delete root `CODEOWNERS`. Confirm `test -f` fails on all four Forgejo paths. | S0 files present on the **same product-PR tip** (not “S0 accepted” alone). Draft delete already on `chore/remove-catchall-codeowners`. |
| **S2** | README rewrite on that **same** tip: state **H15**; relative links to ADR 0001 / architecture (those files already on the tip); do not imply CODEOWNERS or forge `#5` is the trusted-merge gate. No Coolify/Woodpecker YAML edits. Open the leftover issue that will own S3. | S0 files on the same tip as S1. Same PR as S1. |
| **S3** | Leftover-complete: one dated leftover comment with (1) protection GET six flags, (2) dedicated plant-check `{n}` + JSON, (3) four-path `test -f` fails — **H15-1 last**. Does **not** close `#15`. | S0+S1+S2 merged to `main`. Tracked on the leftover issue. |

PR `#15` (or successor) ships **S0+S1+S2**. S2 depends on S0 files being on the same tip, not only on S1.

Sister repos (not slices of #15, not local `DEPS`): forge #48 protection+templates; CAC #429 autoland occupying job.

## Tests

In-repo CI cannot GET branch protection; scanners still must pass on the product PR.

1. **Absence (land, H15-1).** After S1, `test -f CODEOWNERS`, `test -f docs/CODEOWNERS`, `test -f .gitea/CODEOWNERS`, and `test -f .forgejo/CODEOWNERS` all fail. Optional content check, **pathspec those four paths only**: `git grep -nE '^\.\* @' -- CODEOWNERS docs/CODEOWNERS .gitea/CODEOWNERS .forgejo/CODEOWNERS` — no match is pass. Do **not** whole-tree `git grep` (the escaped form `git grep -n '^\\.\\* @'` matches nothing even while the catch-all file exists; the unescaped `git grep -nE '^\.\* @'` hits this ADR after a correct delete).
2. **PR scanners (land).** Existing Woodpecker gitleaks / opengrep / trivy-fs succeed. Coolify step does not run on the PR event.
3. **Protection (leftover-complete, operator read).** Repo admin of `code/hello` attests dated JSON: `GET .../branch_protections` `main` rule equals the six architecture **protection GET** flags. Fail if any differ. Not inferred from a green scanner. Not compared to the Merge API row.
4. **No new plant (leftover-complete).** After the delete is on `main`, on the leftover issue run this recipe:
   1. Open a **dedicated** plant-check PR. PR GET must have `draft == false`. Title must not contain `WIP` (case-insensitive). At least one changed file (any path matches Go `.*`).
   2. Do not request users or teams in the UI or via `POST .../requested_reviewers`.
   3. GET `.../pulls/{n}` and `.../pulls/{n}/reviews` **immediately after open**.
   4. Pass iff Observability’s fail-closed pair holds. If either plant signal is present, fail. If both signals are empty, wait once **30 seconds** and re-GET both; pass only if the second pair is still empty.
   5. Record `{n}` **and** the two JSON bodies (the pair used for the pass decision) on the leftover issue, then **close without merge**.
   Fail if `draft != false`, if the title contains `WIP` (case-insensitive), if the PR has no changed file, if reviewers were requested manually, or if either GET signal is present after the wait. Do not use `#15`. Do not use “the next natural PR.” Who opens the PR: anyone who can create a PR on `code/hello`. Who GETs protection (item 3): repo admin; not Woodpecker.
   Leftover-complete attest order is Integration: **H15-1 last**. Do not treat a passing GET or plant-check from before a later `_ensure_codeowners` apply as done.
5. **Reject still blocks (doc-level).** Do not turn off `block_on_rejected_reviews` to “make autoland easier.”

No unit test harness exists in this repo; do not add one solely for file absence.

## Rollout

- Merge vehicle **B**: existing [#15](https://git.cl8y.com/code/hello/pulls/15) once that branch’s tip is S0 files + S1 delete + S2 README **or** a successor PR with the same three. Design-only `cac-design-issue-15` must not be opened as the product PR and must not be merged first as docs-only.
- Order: protection already live → copy S0 files from the independently accepted SHA + delete file + README via one product PR (land) → leftover issue remains open → leftover-complete as one dated comment with protection GET, plant-check (close without merge), then four-path `test -f` (**H15-1 last**).
- Canary role: other `code/*` catch-all deletions may copy this pattern; this ADR does not merge those repos.
- [#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297): no deploy, spend, custody, or CAC policy expansion. Landing #15 does not authorize Coolify or autonomy changes.

## Rollback

Restore the previous `CODEOWNERS` **via PR**, not direct `main`. That re-plants official requests. It does **not** by itself re-enable merge-block (`block_on_official_review_requests`); restoring the 405 gate is a forge-policy revert, founder-scoped, and is not a hello rollback step.

Woodpecker / Coolify rollback is unused: those files are untouched.

If S0 docs need revert, revert via PR together with README so relative links do not 404.

## Integration completion criteria

### Land (S0+S1+S2) — merge of PR `#15` or successor

All must be true on the merged tip. This is what merging `#15` completes. It does **not** wait for S3.

1. `main` has no CODEOWNERS file at the four Forgejo paths: `test -f` fails on `CODEOWNERS`, `docs/CODEOWNERS`, `.gitea/CODEOWNERS`, `.forgejo/CODEOWNERS` (**H15-1**).
2. Merged tip contains `docs/adr/0001-remove-catchall-codeowners.md` and `docs/architecture.md` **byte-identical** to the independently accepted SHA (this SHA or a successor; not this tip until independent review). README relative links to those paths resolve. Do not merge a README that points at those paths until they exist on that tip.
3. README states the **H15** gate and does not imply CODEOWNERS or forge `#5` is what makes merge trusted. No stub escape.
4. Woodpecker `ci/woodpecker/pr/woodpecker` succeeded on the product PR; deploy secrets did not run on that PR event.
5. No `force_merge`, no direct `main`, no CAC dismiss-as-merge, no Coolify/HMAC/`autonomy.rs` edits in the product-PR diff.
6. A leftover issue exists that owns S3 (because merging `#15` closes that number).

Green scanners on `chore/remove-catchall-codeowners` **before** merge are necessary for land and not sufficient for leftover-complete. Draft `1892075` is incomplete without S0 files and S2.

### Leftover-complete (S3) — follow-up issue; survives merge of `#15`

Require **one dated leftover comment** with all three items, **H15-1 last** (or all three timestamps in one attest). A GET or plant-check from before a later `_ensure_codeowners` apply does not count; re-delete via PR and write a new comment.

1. Protection GET six flags: repo admin of `code/hello` attests dated JSON: `GET .../branch_protections` `main` rule equals the six architecture **protection GET** flags (**H15-2**, **H15-8**, **H15-3**, **H15-9**, **H15-10**, **H15-5**). Not the Merge API row. Not inferred from a green scanner.
2. Plant-check `{n}` + JSON: a **dedicated** plant-check PR following Tests item 4 and Observability’s fail-closed pair. Record `{n}` and the two JSON bodies, then close without merge. `#15`’s own official request does not count. “The next natural PR” does not count.
3. **H15-1 last:** `test -f` fails on `CODEOWNERS`, `docs/CODEOWNERS`, `.gitea/CODEOWNERS`, and `.forgejo/CODEOWNERS`. Recorded after items 1 and 2 (same comment or later timestamp in the same attest).

Forgejo#50 and CAC#429 may stay open; they are not land or leftover gates for this tree.

## Authority

[cl8y-agent-control#297](https://git.cl8y.com/PlasticDigits/cl8y-agent-control/issues/297): this design does not grant deploy, spend, custody, or agent-permission expansion. Relaxing official-review as a **forge merge gate** is already executed on this repo by #48; this ADR only removes the in-tree file that plants requests. Independent review of this proposal is a later gate. Design author must not write `DESIGN: APPROVE`.
