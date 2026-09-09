# Migration

Run the rename pass **before** the first sync against a repo with issue
history. `gh label edit --name` preserves the label on every issue already
carrying it. Delete-and-recreate strips it from all of them, silently — and a
pruning sync deletes anything not in the profiles, so an unrenamed stock label
is a silent loss of issue metadata.

After the renames, the sync only creates what is genuinely new.

## `nodejs-sdk` — the pilot

Current state, sixteen labels: GitHub's stock nine untouched, plus seven
hand-added with **empty descriptions** and unrelated colors.

| Current | Becomes |
|---|---|
| `bug` | `type:bug` |
| `enhancement` | `type:feature` |
| `documentation` | `type:docs` |
| `question` | `type:question` |
| `duplicate` | `resolution:duplicate` |
| `invalid` | `resolution:invalid` |
| `wontfix` | `resolution:wontfix` |
| `core` | `area:core` |
| `resilience` | `area:resilience` |
| `transport` | `area:transport` |
| `adapters` | `area:transport` — **merge**, adapters *are* the transport seam |
| `conformance` | `spec:conformance` |
| `logging` | `area:platform` — logging is `OBS-*` |
| `good first issue` | unchanged, exact string preserved |
| `help wanted` | unchanged, exact string preserved |
| `v1/MVP` | **convert to a milestone**, not a label |

### Commands

Run from a clone of the target repo. Colors and descriptions come from the
profile files, so the sync agrees with the rename and reports no diff.

```sh
gh label edit "bug"           --name "type:bug"              --color 1d76db --description "Defect in shipped behavior"
gh label edit "enhancement"   --name "type:feature"          --color 1d76db --description "New capability or enhancement"
gh label edit "documentation" --name "type:docs"             --color 1d76db --description "Documentation only"
gh label edit "question"      --name "type:question"         --color 1d76db --description "Usage question, not a defect"
gh label edit "duplicate"     --name "resolution:duplicate"  --color cfd3d7 --description "Already tracked elsewhere"
gh label edit "invalid"       --name "resolution:invalid"    --color cfd3d7 --description "Not a defect, or not reproducible"
gh label edit "wontfix"       --name "resolution:wontfix"    --color cfd3d7 --description "Deliberately not doing this"
gh label edit "core"          --name "area:core"             --color 5319e7 --description "Core HTTP, IO, body, context, encoding: HTTP-* IO-* BODY-* CTX-* UTF-*"
gh label edit "resilience"    --name "area:resilience"       --color 5319e7 --description "Retry, recovery, redirects: RETRY-* RECOV-* REDIR-*"
gh label edit "transport"     --name "area:transport"        --color 5319e7 --description "Transport, async model, seams: TRANSPORT-* ASYNC-* SEAM-*"
gh label edit "conformance"   --name "spec:conformance"      --color d4c5f9 --description "Conformance-suite behavior, or a gap in the suite itself"
gh label edit "logging"       --name "area:platform"         --color 5319e7 --description "Config, observability, non-functional, hashing: CFG-* OBS-* NFR-* SHA-*"
```

`good first issue` and `help wanted` are **not** renamed. The exact string is
the interface to GitHub's contributor-discovery surfaces.

### `adapters` — a real merge

Two labels' issues land on one. `gh label edit` cannot rename onto an existing
name, so relabel and then let the prune remove the empty original.

**Read the issues first.** If any distinction between `adapters` and
`transport` matters, this is the moment it is lost.

```sh
gh issue list --label adapters --state all --limit 200 \
  --json number,title --jq '.[] | "\(.number)\t\(.title)"'

for n in $(gh issue list --label adapters --state all --limit 200 --json number --jq '.[].number'); do
  gh issue edit "$n" --add-label "area:transport" --remove-label "adapters"
done
```

The now-unused `adapters` label is deleted by the first pruning sync.

### `v1/MVP` — label to milestone

Release scope, and GitHub already has the right primitive: a milestone gives a
progress bar, a due date, and an open/closed count within the scope. A label
gives none of those.

Create the milestone **before** the pruning run, or the label is deleted and
the membership is gone.

```sh
gh api repos/dexpace/nodejs-sdk/milestones -f title='v1/MVP' -f state=open

for n in $(gh issue list --label "v1/MVP" --state all --limit 200 --json number --jq '.[].number'); do
  gh issue edit "$n" --milestone "v1/MVP"
done
```

The label is then deleted by the first pruning sync.

## The other repos

Same pattern, repo by repo: read the current labels, map the stock nine with
the table above, decide each hand-added label by hand.

```sh
gh label list --limit 100
```

Order per repo: rename, then merge, then milestones, then caller file, then a
dry run, then prune.
