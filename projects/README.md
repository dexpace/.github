# Boards

[Labels](../labels/README.md) answer *what kind of work is this*, across twenty
repositories. A board answers *where is this one, right now*. Different
questions, different instruments — and the vocabulary is split between them
rather than duplicated.

Four primitives, four jobs:

| Instrument | Scope | Holds | Lives |
|---|---|---|---|
| Label | org-wide | what kind of work this is | forever |
| Repo board | one repository | every open issue and PR in it | while the repo has work |
| Campaign board | several repositories | epics only, for one time-boxed push | until the campaign closes, then deleted |
| Milestone | one repository | release scope | until the release ships |

**The tier is set by horizon, never by work type.**

## Shape

GitHub retired classic repository-owned projects. Every repo board is therefore
a **Projects v2 owned by the `dexpace` org and linked to exactly one
repository**, titled with that repository's name, and **private** — a board is
working state, not a published artifact, and that holds for the public repos
too.

**There is no *permanent* org-level board.** Labels stay the only permanent
org-wide layer; standing cross-repo questions are answered by search.

| Question | Answered by |
|---|---|
| What is blocked anywhere in the org | `org:dexpace is:open label:status:blocked` |
| What is awaiting review anywhere | `org:dexpace is:open is:pr -is:draft review:none` |
| What is in flight in `python-sdk` | that repo's board |
| Where the six ports stand on one spec push | that campaign's board |

## Status — four columns, and the empty one

Single-select, one value at a time. **No value is the untriaged state** — the
same rule the labels use, on the other surface.

| Column | Means |
|---|---|
| *(none)* | Not triaged. Arrived and unread. |
| `Todo` | Assessed, actionable, nobody on it yet |
| `In progress` | Someone is working it |
| `In review` | Open pull request awaiting review |
| `Done` | Closed or merged |

**There is no Blocked column**, on purpose. Blocked is orthogonal to position:
blocked work is still *in progress*, and it carries `status:blocked` alongside
whatever column it sits in. A Blocked column would force a false choice and lose
the one fact worth keeping — where the work actually stopped.

`In review` is what makes a board worth having in this org specifically. It is
the only column whose contents are pull requests as the point rather than as a
side effect, and it turns a stalled review into a visible pile rather than a
notification nobody opened.

## Fields — none

Status and nothing else. Every other dimension an item needs is already a label.
Two costs, both accepted:

- **Group-by is Status only.** Projects v2 groups by field, never by label.
  Labels filter; they do not group. A board can *show* only `area:streaming`
  work, but cannot be laid out by area.
- **Impact × effort has no home in GitHub.** With no Priority or Size field,
  that ranking lives in the audit notes. If it must move into GitHub later, the
  cheaper fix is a `priority:` family in `common.yml` — one more dimension the
  whole org can search — not a per-board field that ten boards must agree on.

## Views — two

| View | Layout | Filter |
|---|---|---|
| `Board` | board, grouped by Status | — |
| `Untriaged` | table | `no:status` |

`Untriaged` is the queue the bare-arrival rule creates, and it is the view that
would otherwise be invisible: an item with no Status appears in **no column** of
a board grouped by Status. Without this view, auto-added items silently vanish.
Not a convenience — it is what makes the intake rule survivable.

Blocked work, area breakdowns and per-type slicing are found by typing a label
filter (`label:type:bug`), not by a saved view. Two views is a deliberate floor:
every board is hand-made, and a saved view nobody opens is one more thing to
keep correct.

## Coverage — four open items, or a known inbound backlog

Setup is manual and there is no drift detector, so every board is a standing
cost. A board holding a single item is a worse instrument than the issue list it
duplicates, and an empty one reads as an abandoned one.

| Gets a board now | Open issues + PRs |
|---|---|
| `morphic` | 67 |
| `python-sdk` | 66 |
| `java-sdk` | 54 |
| `spaceapi` | 13 |
| `kuri` | 11 |
| `dexpace-react` | 5 |
| `dotnet-sdk` | 4 |
| `nodejs-sdk` | 2 — the one exception |

**Eight boards, not twelve.** `confluence`,
`dexpace-documentation-archetype`, `ewshotel-api-docs-scalar-test` and
`dexpace.org` hold one open item each and get none. `go-sdk`, `ruby-sdk`,
`styleguide`, `morphic-test-assets`, `zeus`, `devhub`, `devhub-api` and
`.github` have none open at all. All twelve get a board when they cross the
threshold.

`nodejs-sdk` is the single repository below the threshold that gets one anyway,
on two grounds that both hold: it is the rollout pilot for both tracks, and its
backlog is about to arrive — fifteen remediation tasks under umbrella `#67`,
and the audit findings behind them, are drafted and not yet filed. So the rule
is "four open items, **or** a known inbound backlog".

## Provisioning — manual, per repo

No workflow, no cron, no reconciliation, no token. Boards change rarely, and
they cannot be created by a repository's own `GITHUB_TOKEN` — Projects v2 are
org-owned — so automating them would mean holding exactly the org-scoped write
secret the label design refuses to hold.

See [`CHECKLIST.md`](CHECKLIST.md). Run it once per repository.

**Accepted: there is no drift detector.** This is where boards are weaker than
labels, whose pruning weekly sync re-heals whatever the web UI changed. Here the
checklist is the specification, a diverged board is noticed by a person, and it
is fixed by hand. That is the price of not holding an org write token — the same
price the label design pays in a different currency.

## Rejected — a board per work type

The obvious alternative is a board per `type:` value — one for bugs, one for
features — instead of one per repository. Rejected, and the reasons are recorded
because the idea recurs.

- **It splits the wrong axis.** A board is the unit a person holds in their head
  in one sitting, and in this org that unit is a repository. Split by type and
  whoever works `python-sdk` opens two boards to see their own work, neither of
  which shows the whole.
- **Five of the seven types are left homeless.** A bugs board and a features
  board leave `docs`, `test`, `ci`, `chore` and `question` with nowhere to go.
  The endgame is seven boards per repository, or a misc board — and a misc board
  is where work goes to die.
- **It contradicts bare arrival.** A type board needs a type label to route the
  item, and by design no label exists at open, so nothing routes. Every
  repository would then need an inbox board on top, tripling the count of the
  one instrument that has no drift detector.
- **Reclassification costs Status.** Status is a per-project field. A bug that
  turns out to be a feature moves between boards and loses its column;
  relabelling on a single board does not.
- **It buys grouping that filtering already gives.** The pull toward a bugs
  board is really the wish to *group* by type, which Projects v2 cannot do with
  a label. Filtering by type is one string in the filter bar at zero
  provisioning cost, and that is what the wish actually wants nine times in ten.

At eight repo boards, a type split is sixteen or more hand-provisioned boards
for a five-person org — each another place for step 4 of the checklist to be
missed silently.

What the instinct gets right is that a repository is not the only unit of work
here: six ports implement one spec. That is answered by a campaign board.

## Campaign boards — the temporary org-level exception

Org-owned, linked to **several** repositories, existing for one time-boxed push.
The only board allowed to span repositories, and allowed because it has an end:
when the campaign closes, the board is **deleted**. That is what stops it
rotting into an unmaintained org rollup.

| Rule | Why |
|---|---|
| Items are epics, not every issue | A campaign board holding 200 items is a second copy of the repo boards, and the copy goes stale |
| Same Status set, no extra fields | One column vocabulary across every board in the org |
| A named end condition, written into the board description | It is deleted when that condition is met |
| Never auto-add | Auto-add over six repositories drags in everything; campaign items are placed by hand |

Two campaigns earn a board:

| Campaign | Repos | Items | Ends when |
|---|---|---|---|
| **SDK Parity** | the six ports | one epic per `area:` bucket, carrying that bucket's `spec:gap` and `spec:parity` work | the eight buckets are level across the ports |
| **v1 MVP** | the ports that ship v1 | one item per repo `v1/MVP` milestone | every one of those milestones is closed |

`v1 MVP` is a board *over* milestones, not a replacement for them. The milestone
still answers "is this scope finished" inside one repository; the board is the
only place all six answers sit side by side. That is the question the per-repo
design genuinely cannot answer, and it is why the org-level ban is a ban on
*permanent* boards rather than on all of them.

## Rollout order

Boards and labels are independent tracks against the same pilot, `nodejs-sdk`.

1. Build the `nodejs-sdk` board from the checklist. Two open items makes it
   cheap to read end to end.
2. Verify the two things the checklist exists to get right: open a throwaway
   issue, confirm it lands with **no** Status, confirm it appears in
   `Untriaged`. Close it.
3. Build boards for the four heavy backlogs — `morphic`, `python-sdk`,
   `java-sdk`, `spaceapi`. These are where the column set is genuinely tested,
   and where getting it wrong is most expensive to undo.
4. Then `kuri`, `dexpace-react`, `dotnet-sdk` — eight in total — and any other
   repo as it crosses four open items.
5. **SDK Parity** last, and only once at least three ports have a repo board. A
   campaign board over boards nobody has populated yet is a board of guesses.
   `v1 MVP` waits on the milestones existing.

Board setup is reversible in a way the label prune is not: deleting a project
loses its Status values and nothing else. No issue, pull request, label or
milestone is touched. That asymmetry is why the boards can go first and the
prune cannot.
