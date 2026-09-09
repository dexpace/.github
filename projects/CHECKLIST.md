# Board provisioning checklist

Run once per repository. Everything here is web UI — Projects v2 are org-owned,
so none of it can be done by a repository's own `GITHUB_TOKEN`, and automating
it would mean holding the org-scoped write secret the label design refuses to
hold.

Rationale for every choice below is in [`README.md`](README.md).

---

- [ ] **1. New project** at `github.com/orgs/dexpace/projects`, **Board**
      template. Title it the repository name, nothing else.

- [ ] **2. Link the repository** — project *Settings → Linked repositories* —
      and only that one.

- [ ] **3. Status options** — rename to exactly `Todo`, `In progress`,
      `In review`, `Done`. **Delete any default option not on that list**
      (`Backlog` ships enabled and is not on it).

- [ ] **4. Workflows → `Item added to project`** — enable, scope to issues
      **and** pull requests, and **clear its `Set value → Status: Todo`
      action**.

- [ ] **5. Workflows → `Item closed`** — enable, set Status to `Done`.

- [ ] **6. Views** — rename the default view to `Board`, grouped by Status.
      Add a **table** view named `Untriaged`, filtered `no:status`.

- [ ] **7. Visibility** — leave **private**.

---

## Step 4 is the one not to skip

It is the only step whose omission is **invisible**. A board with the Todo
default left on looks completely correct and is quietly wrong, because nothing
is ever untriaged — an unread issue and an assessed one become indistinguishable
forever, and no later inspection recovers which was which.

GitHub ships that action **on**. Turning it off is the board's exact equivalent
of the issue-template `labels:` trap: a default that quietly classifies things
for you.

Auto-add itself is fine and is deliberately kept. It *places* an item; it
*assigns* nothing. Bare arrival survives on both surfaces — an item lands on the
board with no Status exactly as it lands in the issue list with no label.

## Verify before calling a board done

Two checks, and they are the whole point of the checklist:

1. Open a throwaway issue in the linked repo. It must appear on the board with
   **no Status** — in no column of the `Board` view.
2. It must appear in the `Untriaged` view.

Close it.

If it landed in `Todo`, step 4 was missed. Fix it, then delete the Status value
on every item that got one by accident — there is no way to tell later which
were triaged.

## Not automated, on purpose

No workflow, no cron, no reconciliation, no token, and therefore **no drift
detector**. The checklist is the specification, a diverged board is noticed by a
person, and it is fixed by hand.

Steps 4 and 5 could not be scripted even if the policy allowed it: Projects v2
built-in workflows have no public GraphQL mutation.
