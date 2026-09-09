# Labels

Org-wide label vocabulary for `dexpace`, defined once here and segregated by
**profile** so SDK-only labels never reach a tool repo.

Labels answer *what kind of work is this*, across twenty repositories.
[Boards](../projects/README.md) answer *where is this one, right now*. The
vocabulary is split between them rather than duplicated.

## Profiles

| File | Scope | Labels |
|---|---|---|
| [`common.yml`](common.yml) | every repository | 20 |
| [`docs.yml`](docs.yml) | repos where documentation is tracked work | 7 |
| [`package.yml`](package.yml) | repos that publish a versioned artifact | 2 |
| [`sdk.yml`](sdk.yml) | the six ports only | 13 |

| Repos | Profiles | Total |
|---|---|---|
| The six SDK ports | `common,docs,package,sdk` | 42 |
| `kuri`, `morphic` | `common,docs,package` | 29 |
| `styleguide`, `morphic-test-assets`, `confluence`, `devhub`, `dexpace.org`, `.github`, the two doc archetypes | `common,docs` | 27 |
| `zeus`, `dexpace-react` | `common,package` | 22 |
| `spaceapi`, `devhub-api` | `common` | 20 |

Forty-two is large because it is four independent dimensions rather than one
flat list — a given issue carries roughly three of them, not forty. Candidates
to cut first if the vocabulary turns noisy: `performance`, `regression`,
`docs:site`, `compat:runtime`, `release:versioning`. Each is a one-line deletion
and a re-sync.

`package` is deliberately not folded into `sdk`. `kuri` is the only repository
in the org with published releases and it is a Kotlin URI library, not a port.

## `area:*` — the normative mapping

**This table is normative.** Nobody reasons about which bucket a requirement
belongs to; they look it up here. An issue citing `SSE-42` is `area:streaming`,
always — the fact that it describes reconnect behavior never pulls it toward
`area:resilience`.

| Label | ID families | Spec chapters |
|---|---|---|
| `area:core` | `HTTP-*` `IO-*` `BODY-*` `CTX-*` `UTF-*` | 4, 5, 6, 7 |
| `area:pipeline` | `PIPE-*` `XCUT-*` | 8, 19 |
| `area:resilience` | `RETRY-*` `RECOV-*` `REDIR-*` | 9, 10 |
| `area:auth` | `AUTH-*` | 11 |
| `area:streaming` | `SSE-*` `PAGE-*` | 12, 13 |
| `area:serde` | `SERDE-*` | 14 |
| `area:transport` | `TRANSPORT-*` `ASYNC-*` `SEAM-*` | 3, 17, 18 |
| `area:platform` | `CFG-*` `OBS-*` `NFR-*` `SHA-*` | 15, 16, 20 |

Every family maps to exactly one bucket, and every bucket has at least one
family. Adding a family to the spec means **adding a row here**, not inventing
a label.

## How a repository gets these labels

Pull, never push. In the target repo: **Actions → New workflow → Labels**,
under *Workflows created by dexpace*. Or copy
[`.github/workflow-templates/labels.yml`](../.github/workflow-templates/labels.yml)
in by hand as `.github/workflows/labels.yml`. Then set `profiles`. That one file
is the entire per-repo cost.

The caller runs in that repo with the `GITHUB_TOKEN` GitHub already grants the
run, and mutates only that repo. No org PAT exists anywhere. A push model would
need a token able to rewrite every repository in the org, held as a secret.

**Nothing here propagates on its own.** A repo without the caller file is
untouched.

### Onboarding a repository

The order is not cosmetic. **Renames must happen before the caller file
exists.** If a sync creates `type:bug` first, `gh label edit bug --name
type:bug` fails because the name is taken, and the cheap migration — which
preserves the label on every issue carrying it — is gone for good.

The template ships dispatch-only for exactly this reason: committing it cannot
fire a sync.

1. **Rename pass** — [`MIGRATION.md`](MIGRATION.md). Before anything else.
2. **Add the caller**, set `profiles`. Nothing runs.
3. **Dispatch, `dry_run` on.** Read the plan. This is the review step, and it
   is the only one that is free.
4. **Dispatch, `dry_run` off.** Labels are created and corrected. Still no
   deletions — `prune` defaults off.
5. **Set repository variable `LABELS_PRUNE=true`**, dispatch once more with
   `prune` on. Strays are deleted. This is the irreversible moment; step 3
   is what makes it reviewable.
6. **Uncomment the `push` and `schedule` triggers** in the caller.

Step 6 is the one that gets forgotten. Without the cron, a label edited in the
web UI stays edited and the YAML quietly becomes advisory. Steps 1 and 6 are
the two ends of the onboarding, and both are silent when skipped.

### Deletion is opt-in per repository

Only a `workflow_dispatch` that explicitly asks for it, or a repository with
`LABELS_PRUNE=true`, can delete a label. An unattended cron in a repo without
that variable heals colors and descriptions and restores labels someone
deleted by hand, but removes nothing.

That is deliberate. Prune is what makes these files authoritative, and it is
also what destroys issue metadata when a repo was onboarded carelessly. Making
the destructive half a separate, visible switch costs one click per repo.

### Labels Dependabot owns

Dependabot applies an ecosystem label alongside `dependencies` — `javascript`
in `nodejs-sdk`, `python`, `go`, `ruby`, `nuget` elsewhere — and recreates it
whenever it goes missing. Pruning those starts a fight that repeats every
Monday.

They cannot be declared in `common.yml` because the ecosystem differs per
repository, so the workflow's `exclude` input holds them instead and defaults
to the set this org needs. `dependencies` itself is different: it is identical
everywhere, so it is declared in `common.yml` rather than excluded.

## Adding or changing a label

Edit the profile file. That is the only place — **pruning is what makes these
files authoritative**, so a label hand-created in the web UI is deleted on the
next sync. That is the point, not a bug.

Two constraints when editing:

- Colors are hex **without** `#`.
- Descriptions are capped at 100 characters by GitHub.

## Two things that look like mistakes and are not

**`good first issue` and `help wanted` are unprefixed.** GitHub's own
contributor-discovery surfaces match them by exact string. There is no prefixed
equivalent; the spelling is the interface. `meta:good-first-issue` would
silently drop the six public ports out of that discovery path.

**`dependencies` is declared even though nobody applies it by hand.** Dependabot
creates that exact label itself when it is missing, and a pruning sync would
then delete and re-create it every Monday. Declaring it ends the fight.

## What is deliberately absent

- **No `status:` label for flow position.** `status:accepted` and
  `status:in-progress` are board columns. A label that duplicates a column
  drifts the first time someone drags a card without relabelling.
- **No `status:in-review`.** GitHub answers that natively, retroactively, and
  correctly on day one:
  ```
  org:dexpace is:open is:pr -is:draft review:none
  ```
  A label version reports the habit of applying labels rather than the state of
  the review queue — and on a team measured at zero reviews on 136 merged PRs,
  that habit is the one thing that cannot be assumed.
- **No `status:triage`.** Nothing applies a label on creation, so **bare is the
  untriaged state** and needs no label of its own. A `status:triage` label would
  only ever be applied by hand to something already sitting in the state it
  describes.
- **No labeler workflow, no auto-triage bot.** If an issue template is ever
  added, it ships an empty `labels:` field — a YAML issue form's `labels:` key
  applies its contents silently on submit, and a template added months from now
  is the most likely way this decision gets undone by accident.

## Known trade-off

"Which repos are SDKs" lives in the caller files, not one central manifest.
That is the cost of not holding an org-wide write token. If the drift ever
bites, the fix is a `labels/profiles.yml` here that the reusable workflow reads
by repository name — the callers stay as they are and just stop passing
`profiles`.
