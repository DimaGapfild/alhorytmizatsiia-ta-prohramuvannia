# Project 5. To-Do List

| | |
|---|---|
| **Type** | console application (command line) |
| **Difficulty** | ★★☆☆☆ |
| **Chapters used** | 7 (strings), 8 (lists), 10 (functions), 11 (sorting), 12 (dictionaries), 13 (files, errors), 14 (OOP) |
| **Ticket prefix** | `TODO` |
| **Team** | 4–5 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "I keep forgetting my homework and errands. Paper lists get lost, and
> the apps on my phone have too many buttons. I want a tiny program in
> the terminal: I type a task, I see my list, I mark what is done. And it
> must not forget my tasks when I close it!"

| Question | Answer |
|----------|--------|
| Do tasks have deadlines? | Some do. It would be nice to see what is overdue. |
| Priorities? | Yes: high, normal, low. |
| Should done tasks disappear? | No, I like to see what I've done. But hide them if I ask. |
| Menu or commands? | Commands, like `add Buy milk`. A `help` command please. |

---

## Stage 1. Product Owner — Product Description

### Vision

A **simple, fast** to-do list in the terminal. The user types short
commands; the tasks are saved to a file automatically. No account, no
internet, no clutter.

### Users

- **Student** — homework and deadlines.
- **Busy person** — short errands, wants to add a task in one line.

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a user, I want to add a task with one command, so that it is quick. | Must |
| US-2 | As a user, I want to see all my tasks with numbers, so that I can refer to them. | Must |
| US-3 | As a user, I want to mark a task as done (and undo it), so that I see my progress. | Must |
| US-4 | As a user, I want to delete a task. | Must |
| US-5 | As a user, I want my tasks to be saved automatically, so that nothing is lost. | Must |
| US-6 | As a user, I want a `help` command and clear error messages. | Must |
| US-7 | As a user, I want to give a task a priority and see high-priority tasks first. | Should |
| US-8 | As a student, I want to set a due date and see overdue tasks highlighted. | Should |
| US-9 | As a user, I want to filter: only open / only done / only overdue. | Should |
| US-10 | As a user, I want to edit the text of a task. | Could |
| US-11 | As a user, I want to search tasks by a word. | Could |
| US-12 | As a user, I want to sync tasks with my phone. | Won't |

### Acceptance criteria (examples)

**US-1**
- `add Buy milk` → task 1 "Buy milk", priority normal, no due date.
- `add Physics lab !high @2026-12-01` → priority high, due 2026-12-01.
- `add` with no text → "The task text cannot be empty."

**US-2** — the list format:

```
#  St  Pri   Due         Task
1  [ ] HIGH  2026-12-01  Physics lab
2  [x] norm  —           Buy milk
3  [ ] low   2026-11-01  Return the book   (OVERDUE)
```

**US-5**
- After `add` and closing the program, the task is still there when the program starts again.
- If `tasks.json` is broken, the program renames it to `tasks.broken.json`, starts with an empty list and tells the user. **It never silently deletes data.**

**Task numbers:** the number is a stable **ID**, not the position in the
list. After deleting task 2, task 3 is still number 3. (The PO and the
TL must agree on this early — it changes the code a lot.)

### MVP scope

**MVP = US-1 … US-6.** Release goal: + US-7, US-8, US-9.

---

## Stage 2. Project Manager — Project Plan

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup | — | repo, board, command list agreed between PO and TL |
| 1 | "A list that remembers" | US-1, US-2, US-5, US-6 | tasks survive a restart |
| 2 | "Done and gone" | US-3, US-4, US-7 | **MVP** → Staging |
| 3 | "Deadlines" | US-8, US-9 (+ US-10, US-11) | **Release v1.0.0** |

### Risks

| Risk | Plan |
|------|------|
| Saving in the middle of writing the file → a broken file | write to `tasks.json.tmp`, then rename (atomic save) |
| Dates are hard (formats, "today") | TL: only `YYYY-MM-DD`, `datetime.date.fromisoformat`; "today" is a parameter for tests |
| Command parsing grows into a mess of `if`s | TL: a dictionary "command → function" |

---

## Stage 3. Tech Lead — Technical Overview

### Layers

```
todo/
├── main.py         # the REPL: read a line → run a command → print
├── commands.py     # COMMANDS = {"add": cmd_add, "list": cmd_list, ...}
├── models.py       # class Task, class TaskList
├── storage.py      # load/save JSON, atomic save, broken-file handling
├── parser.py       # "Physics lab !high @2026-12-01" → text, priority, due
└── tests/
```

```
main.py ──> commands.py ──> models.py
                 │
                 └──> storage.py
```

`models.py` does not know about files or the console.

### Data model

```python
class Task:
    id: int
    text: str
    done: bool
    priority: str          # "high" | "normal" | "low"
    due: date | None
    created: str           # ISO date

class TaskList:
    tasks: dict[int, Task] # id → task
    next_id: int
    add(text, priority, due) -> Task
    get(id) -> Task        # KeyError → "No task with ID 7"
    remove(id)
    sorted_view(filter, today) -> list[Task]
```

`tasks.json`:

```json
{"next_id": 4,
 "tasks": [{"id": 1, "text": "Physics lab", "done": false,
            "priority": "high", "due": "2026-12-01", "created": "2026-11-13"}]}
```

### Sorting (Chapter 11)

Sort key: `(task.done, PRIORITY_ORDER[task.priority], task.due or date.max, task.id)` —
open tasks first, then by priority, then the nearest due date, then by ID.

### Commands

```
add <text> [!high|!low] [@YYYY-MM-DD]
list [all|open|done|overdue]
done <id>      undo <id>      delete <id>
edit <id> <new text>
find <word>
help           quit
```

### Language notes

- **C++:** JSON is not in the standard library — use a simple text
  format (one task per line, fields separated by `|`) and escape `|` in the text.
- **JavaScript:** `fs.readFileSync` / `JSON.parse`; the CLI with `readline`.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| TODO-1 | Repo, structure, command list in README | — | 1 | — | 0 |
| TODO-2 | `Task` and `TaskList` (add, get, remove, stable IDs) | US-1, US-4 | 3 | TODO-1 | 1 |
| TODO-3 | `parser.py`: text, `!priority`, `@date`, errors | US-1, US-7, US-8 | 3 | TODO-1 | 1 |
| TODO-4 | REPL + command dictionary + `help` + unknown command | US-6 | 2 | TODO-1 | 1 |
| TODO-5 | `add` and `list` commands, table format | US-1, US-2 | 2 | TODO-2, TODO-4 | 1 |
| TODO-6 | `storage.py`: load/save JSON, atomic save | US-5 | 3 | TODO-2 | 1 |
| TODO-7 | Broken / missing file handling | US-5 | 2 | TODO-6 | 1 |
| TODO-8 | `done`, `undo`, `delete` with ID validation | US-3, US-4 | 2 | TODO-5 | 2 |
| TODO-9 | Priority sorting in `list` | US-7 | 2 | TODO-5 | 2 |
| TODO-10 | Unit tests: models, parser, storage | US-1…7 | 3 | TODO-3, TODO-7 | 2 |
| TODO-11 | Overdue detection and mark | US-8 | 2 | TODO-9 | 3 |
| TODO-12 | Filters `open/done/overdue` | US-9 | 2 | TODO-11 | 3 |
| TODO-13 | `edit` and `find` | US-10, US-11 | 2 | TODO-8 | 3 |
| TODO-14 | README with examples, release notes | — | 1 | TODO-8 | 3 |

**MVP total:** 25 SP (TODO-1 … TODO-10).

---

## Stage 5. Testing

### Unit tests

```python
from datetime import date
from models import TaskList
from parser import parse_add
from storage import save, load

def test_ids_are_stable():
    tl = TaskList()
    a, b, c = tl.add("a"), tl.add("b"), tl.add("c")
    tl.remove(b.id)
    assert tl.get(c.id).text == "c"
    assert tl.add("d").id == 4        # IDs are never reused

def test_parse_add():
    assert parse_add("Lab !high @2026-12-01") == ("Lab", "high", date(2026, 12, 1))
    assert parse_add("Buy milk") == ("Buy milk", "normal", None)

def test_parse_bad_date():
    try:
        parse_add("Lab @2026-13-45")
        assert False
    except ValueError:
        pass

def test_overdue():
    tl = TaskList()
    t = tl.add("old", due=date(2026, 11, 1))
    view = tl.sorted_view("overdue", today=date(2026, 11, 13))
    assert view == [t]

def test_save_and_load(tmp_path):         # pytest gives a temporary folder
    path = tmp_path / "tasks.json"
    tl = TaskList(); tl.add("x")
    save(tl, path)
    assert load(path).get(1).text == "x"
```

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | First start (no file) | empty list, "No tasks yet. Type `help`." |
| TC-2 | `add Buy milk`, `list` | task 1 shown |
| TC-3 | `add` | "The task text cannot be empty." |
| TC-4 | `done 1`, `list` | `[x]` for task 1 |
| TC-5 | `done 99`, `done abc` | "No task with ID 99", "ID must be a number" |
| TC-6 | `delete 1`, `add New`, `list` | the new task has ID 2, not 1 |
| TC-7 | Add tasks, quit, start again | all tasks are there |
| TC-8 | Write `{{{` into `tasks.json`, start | message about a broken file; `tasks.broken.json` exists |
| TC-9 | `add Lab @2026-02-30` | "Invalid date" |
| TC-10 | Add a task with a due date in the past, `list overdue` | it is shown with `(OVERDUE)` |
| TC-11 | `add Café ☕ notes` | Unicode saved and shown correctly |

---

## Stage 6. Staging

1. `v1.0.0-rc1`; code freeze; fresh clone.
2. QA runs TC-1 … TC-11 and the unit tests.
3. **UAT:** the PO uses the app for their **real tasks** during the
   staging phase and writes down everything that feels inconvenient.
   Inconvenience is not a bug — it goes into the backlog as a story.
4. **Data safety check:** QA kills the program (Ctrl+C) in the middle of
   work 10 times — the file must never be broken.

### Staging checklist

- [ ] fresh clone works; `help` lists every command
- [ ] all unit tests pass
- [ ] broken-file handling never loses data
- [ ] PO accepted the Must stories

---

## Stage 7. Production

1. PR `staging → main`, tag `v1.0.0`, GitHub Release.
2. **Dogfooding:** the other teams use your app for their own project
   tasks; their feedback goes into your backlog.
3. Hotfix if needed → `v1.0.1`. Retro.

### Release notes (example)

```
## To-Do v1.0.0

New:
  - add / list / done / undo / delete / edit / find
  - priorities (!high, !low) and due dates (@YYYY-MM-DD)
  - overdue tasks are marked; filters open/done/overdue
  - tasks are saved automatically to tasks.json
Known issues:
  - none
How to run:
  python3 main.py
```

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Projects** | "I want to group tasks: 'School', 'Home'." — data format changes! TL: how do old `tasks.json` files keep working? (a **migration**) |
| **Data loss** | "I lost all my tasks after the update!" — critical production bug → hotfix + post-mortem. |
| **One-liner** | "I want `python3 main.py add Buy milk` to work without opening the app." — `sys.argv`; small story. |
| **Unclear story** | "Delete should ask 'Are you sure?'" — but the TL's tests expect no question. PO decides. |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** MVP works from a fresh clone; tasks survive a
restart; a broken file does not crash the app.
