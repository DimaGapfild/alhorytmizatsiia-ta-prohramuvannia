# Projects

Larger tasks that combine material from several chapters. Each project
can be solved in **Python 3** (main language), **C++** or **JavaScript**.

There are two kinds of projects:

- **Classic project** — an individual task with a list of requirements.
- **Team project (the development flow game)** — a team of 4–6 students
  takes a product through the whole real-life flow: *idea → Product Owner
  → Project Manager → Tech Lead → tasks → development → testing → staging
  → production*. Each student plays a role (PO, PM, TL, Developer, QA).

> Before starting any team project, read
> **[The Development Flow Game](projects/00-dev-flow.md)** — roles,
> stages, git workflow, board, templates, event cards and grading.

---

## List of Projects

| No. | Project | Kind | Type | Difficulty | Chapters |
|-----|---------|------|------|------------|----------|
| 1 | [A Simple Notification System](projects/01-notification-system.md) | classic | console app | ★★★☆☆ | 10, 12, 13, 14 |
| 2 | [Memory — Find the Pair](projects/02-memory-cards.md) | team | game | ★★☆☆☆ | 5, 6, 8, 9, 10, 13 |
| 3 | [Sea Battle](projects/03-sea-battle.md) | team | game | ★★★★☆ | 5, 6, 8, 9, 10, 12, 13, 14 |
| 4 | [Snake](projects/04-snake.md) | team | game | ★★★☆☆ | 5, 6, 8, 9, 10, 12, 13, 15 |
| 5 | [To-Do List](projects/05-todo.md) | team | app | ★★☆☆☆ | 7, 8, 10, 11, 12, 13, 14 |
| 6 | [Notes with Tags and Search](projects/06-notes.md) | team | app | ★★★☆☆ | 7, 8, 10, 11, 12, 13, 14, 15 |
| 7 | [Personal Budget (Expense Tracker)](projects/07-expense-tracker.md) | team | app | ★★★☆☆ | 4, 5, 7, 10, 11, 12, 13, 14 |

**Suggested order:** start with an easy project (2 or 5) to learn the
flow, then take a harder one with **rotated roles**.

---

## What Every Team Project File Contains

| Stage | Section in the file | Main role |
|-------|---------------------|-----------|
| 0 | The idea from the Client + questions and answers | Client (instructor) |
| 1 | Product description: vision, users, user stories, acceptance criteria, MVP | Product Owner |
| 2 | Project plan: sprints, milestones, roles, risks | Project Manager |
| 3 | Technical overview: modules, data structures, algorithms, code rules | Tech Lead |
| 4 | Task list: tickets with estimates and dependencies | PM + TL |
| 5 | Testing: unit tests, manual test cases, test report | QA |
| 6 | Staging: release candidate, checklist, PO acceptance | QA + PO |
| 7 | Production: release, release notes, hotfixes, retro | PM + TL |
| — | Event cards: surprises from the Client | Game Master (instructor) |

The documents in each project are a **reference example**. The
instructor decides how to use them:

- **Easy mode** — the team gets the whole file and follows it.
- **Normal mode** — the team gets only the **idea** (Stage 0); each role
  writes their own artifact, then compares it with the example.
- **Hard mode** — normal mode + 2 event cards during development.
