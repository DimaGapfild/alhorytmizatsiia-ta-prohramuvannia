# The Development Flow Game

These projects are a **role-play of a real software team**. You do not
just write a program — you take a product from an idea to a public
release, the way it happens in a real company:

```
 Idea → Product Owner → Project Manager → Tech Lead → Tasks
      → Development → Testing → Staging → Production
```

Every stage produces an **artifact** (a document, a board, a branch, a
release). The next stage can start only when the previous artifact is
ready. The instructor plays the **Client** (the person who had the idea)
and the **Game Master**, who sometimes plays an **event card** — a
surprise that happens in real projects too.

---

## 1. The Team and the Roles

A team has **4–6 students**. Every student writes code, but each one
also has one main role. In the next project the roles **rotate**, so by
the end of the course everyone has tried several of them.

| Role | Responsible for | Main artifacts |
|------|-----------------|----------------|
| **Client** (instructor) | the idea, answers questions about "what is needed" | the idea text |
| **Product Owner (PO)** | *what* we build and *why*; priorities; accepts the result | product description, user stories, acceptance criteria |
| **Project Manager (PM)** | *when* and *who*; the board, sprints, risks, meetings | sprint plan, board, risk list, sprint reports |
| **Tech Lead (TL)** | *how* we build it; architecture, code rules, code review | technical overview, module structure, review comments |
| **Developer (Dev)** | writes the code for tickets, writes unit tests | branches, pull requests, code |
| **QA Engineer (QA)** | checks that the product works as described | test plan, test cases, bug reports, test report |

In a team of 4: PO + PM can be one person, TL + QA cannot (nobody tests
their own architecture alone). In a team of 6: two developers.

> **Rule:** the PO never tells developers *how* to write code, and the TL
> never decides *what* features the product has. When they disagree,
> the PM brings both to the table and they agree on a compromise.

---

## 2. The Stages

| # | Stage | Who | Input | Output (artifact) | Gate to the next stage |
|---|-------|-----|-------|-------------------|------------------------|
| 0 | **Idea** | Client | — | a short text in plain words | the team has read it and asked questions |
| 1 | **Product description** | PO | idea + answers from the Client | vision, users, user stories, MVP scope | the Client agrees: "yes, this is what I want" |
| 2 | **Project plan** | PM | product description | sprints, roles, board, risks | the team agrees with the plan |
| 3 | **Technical overview** | TL | product description | modules, data structures, algorithms, code rules | every developer understands where their code goes |
| 4 | **Task list** | PM + TL | plan + tech overview | tickets with estimates and dependencies | every MVP story is covered by tickets |
| 5 | **Development** | Devs | tickets | feature branches, pull requests | PR reviewed and merged, unit tests pass |
| 6 | **Testing** | QA | merged code + stories | test cases, bug reports, test report | no open **critical** or **major** bugs |
| 7 | **Staging** | QA + PO | release candidate | staging checklist, PO acceptance (UAT) | the PO signs: "accepted" |
| 8 | **Production** | PM + TL | accepted release candidate | tag `v1.0.0`, release notes, demo | another team installs and uses the product |
| 9 | **Retrospective** | everyone | the whole project | what went well / badly / what to change | — |

Stages 5 and 6 repeat in every sprint: develop → test → fix → develop.

---

## 3. Environments for a Student Project

Real companies run the same program in three places. We copy the idea
with git:

| Environment | Where it lives | Who uses it | Rules |
|-------------|----------------|-------------|-------|
| **Development** | `feature/...` branches on each developer's computer | the developer | anything goes; break things freely |
| **Staging** | the `staging` branch, run from a **fresh clone** | QA and PO | only merged and reviewed code; only bug fixes after the "code freeze" |
| **Production** | the `main` branch + a git tag `v1.0.0` + a GitHub Release | "real users" (another team) | nothing is changed directly; only through a release or a hotfix |

Why a *fresh clone* for staging? Because "it works on my computer" is the
most famous bug in the industry. If the program does not start from a
clean folder by following the README — it is not ready.

---

## 4. Git Workflow

```
main ─────────●────────────────────●──── (v1.0.0)   (v1.0.1)
               \                  /  \               /
staging ────────●────●────●──────●    \── hotfix/... ─
                 \   ↑    ↑
feature/MEM-3 ────●──┘    │
feature/MEM-5 ────────●───┘
```

- `main` — production. Changes come **only** from `staging` (a release)
  or from `hotfix/...` branches.
- `staging` — the integration branch. All feature branches are merged here.
- `feature/<TICKET-ID>-short-name` — one branch per ticket, for example
  `feature/SNK-4-food-spawn`.
- `hotfix/<TICKET-ID>-short-name` — urgent fixes for production; merged
  into **both** `main` and `staging`.

**Commit messages** start with the ticket ID:

```
SNK-4: spawn food on a random empty cell
SNK-4: add unit test for a full board
```

**Pull requests:** at least **one approval** (the TL or another
developer). The author never merges their own PR without a review.

---

## 5. The Board

The PM keeps the board (GitHub Projects, Trello, or a table in
`BOARD.md`):

```
Backlog → To Do (this sprint) → In Progress → Code Review → Testing → Done
```

- A developer has **at most 2 tickets** "In Progress" at the same time.
- A ticket goes to **Done** only when it meets the Definition of Done.

### Definition of Ready (a ticket can be taken into a sprint)

- it has a clear title and a description;
- it has acceptance criteria;
- it has an estimate in story points;
- its dependencies are done or planned earlier in the same sprint.

### Definition of Done (a ticket is finished)

- the code is merged into `staging` after a review;
- unit tests for the new logic exist and pass;
- the acceptance criteria are checked by QA;
- the code follows the TL's code rules (names, docstrings, no dead code);
- the README is updated if the behaviour visible to the user changed.

### Estimates

We estimate **complexity, not time**, in story points from the
Fibonacci row: **1, 2, 3, 5, 8**. A ticket bigger than 8 must be split.
The PM sums the points of a sprint; after Sprint 1 the team knows its
**velocity** (how many points it really finishes per sprint) and plans
the next sprints with it.

---

## 6. Templates

### User story (PO)

```
US-3. As a <type of user>, I want <action>, so that <benefit>.

Acceptance criteria:
  - Given <situation>, when <action>, then <result>.
  - ...
Priority: Must / Should / Could / Won't
```

### Ticket (PM + TL)

```
ID:          SNK-4
Title:       Spawn food on a random empty cell
Story:       US-2
Estimate:    3 SP
Depends on:  SNK-2
Description: ...
Acceptance:  - food never appears on the snake
             - if the board is full, the game is won
```

### Bug report (QA)

```
ID:          BUG-7
Title:       The game crashes when the player enters "Z9"
Severity:    critical / major / minor / trivial
Environment: staging, commit 3f2a1c9, Python 3.12, Windows 11
Steps:       1. Start the game  2. Enter Z9  3. Press Enter
Expected:    the message "Invalid cell, try again"
Actual:      IndexError: list index out of range
Related:     US-4, MEM-6
```

**Severity:**

| Severity | Meaning | Can we release? |
|----------|---------|-----------------|
| critical | crash, data loss, the main scenario cannot be finished | no |
| major | a Must story works wrongly, no workaround | no |
| minor | wrong behaviour with a workaround, or in a Should story | yes, listed in "Known issues" |
| trivial | a typo, alignment, colour | yes |

### Pull request description (Dev)

```
Ticket: SNK-4
What:   food spawns on a random empty cell
How:    collect empty cells into a list, then random.choice()
Tests:  test_food_not_on_snake, test_full_board_is_win
```

### Release notes (PM)

```
## v1.0.0

New:
  - ...
Fixed:
  - ...
Known issues:
  - BUG-9 (minor): ...
How to run:
  python3 main.py
```

### Retrospective (everyone)

Each member writes 1–3 notes in every column; the team then picks
**one** thing to change in the next project.

| Went well | Went badly | Change next time |
|-----------|------------|------------------|

---

## 7. Event Cards (Game Master)

During development the instructor may play **one or two** event cards.
Every project file has its own cards; these are the general ones:

| Card | What happens | What the team must do |
|------|--------------|-----------------------|
| **Change request** | the Client wants a new feature | PO decides: put it into the backlog or replace a lower-priority story; PM updates the plan |
| **Sick day** | one developer cannot work this sprint | PM moves their tickets; the team decides what to cut |
| **Production bug** | a "user" finds a bug in `v1.0.0` | QA writes a bug report → `hotfix/...` branch → `v1.0.1` |
| **Scope cut** | the Client says: "release earlier, with less" | PO cuts Should/Could stories, keeps the MVP working |
| **Unclear story** | a story turns out to have two meanings | Dev stops and asks the PO; PO updates the acceptance criteria |
| **Code review war** | the TL rejects a PR twice | the author and the TL meet and agree; the PM records the decision |

The point of the cards is not to punish the team — it is to practise
the reaction. A team that handles a card well gets bonus points.

---

## 8. Grading

The team gets a common grade for the product and every student gets a
personal grade for their role.

**Team grade (1–5):**

| Grade | Requirements |
|-------|--------------|
| 5 | all stages done with artifacts; all Must stories released; tests exist; release notes and retro written |
| 4 | all stages done; small gaps in artifacts or a minor bug in a Must story |
| 3 | the MVP works and is released, but some artifacts are missing (e.g. no test report or no staging check) |
| 2 | the program works partly; the flow was skipped (code without stories or tickets) |

**Personal grade:** the instructor checks the artifacts of the student's
role (stories for the PO, the board for the PM, the overview and the
reviews for the TL, merged PRs for a Dev, bug reports for QA) and the
student's commits. **Bonus +1** (up to 5) for handling an event card
well.
