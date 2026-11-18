# Project 4. Snake

| | |
|---|---|
| **Type** | console game (turn-based MVP, real-time as a bonus) |
| **Difficulty** | ★★★☆☆ |
| **Chapters used** | 5, 6, 8 (lists), 9 (matrices), 10 (functions), 12 (sets, tuples), 13 (files), 15 (complexity) |
| **Ticket prefix** | `SNK` |
| **Team** | 4–5 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "The snake game from old phones! The snake moves, eats food, gets
> longer, and you lose if it bites itself or hits the wall. I want to
> play it in the terminal and to have a high score table."

| Question | Answer |
|----------|--------|
| Must the snake move by itself, or step by step? | By itself would be great… but if that is too hard, step by step is OK for the first version. |
| Walls or no walls? | Walls. Maybe a "no walls" mode later. |
| A high score table — for one person or many? | Top 5 with names. |

> **Note for the TL:** the Client's first answer is a classic trade-off.
> Real-time input in a console is platform-specific. A good TL proposes:
> "MVP is turn-based (the snake moves one step per key press), real-time
> is a Should story with `curses`". The PO must agree — this is the first
> negotiation of the project.

---

## Stage 1. Product Owner — Product Description

### Vision

A console Snake game. The player controls the snake with `W A S D`,
eats food to grow and earn points, and tries to get into the top-5
table. The first release is **turn-based**; the real-time mode comes next.

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a player, I want to see the field with walls, the snake and the food. | Must |
| US-2 | As a player, I want to control the snake with `W A S D` (Enter alone = keep the direction). | Must |
| US-3 | As a player, I want the snake to grow by one cell and get +10 points when it eats food. | Must |
| US-4 | As a player, I want new food to appear on a free cell after the old one is eaten. | Must |
| US-5 | As a player, I want the game to end when the snake hits a wall or itself, and see my score. | Must |
| US-6 | As a player, I want the snake not to turn back into itself (pressing `S` while moving up is ignored). | Must |
| US-7 | As a player, I want a top-5 high score table with names, saved between games. | Should |
| US-8 | As a player, I want a real-time mode where the snake moves by itself. | Should |
| US-9 | As a player, I want the speed to grow with the score (real-time mode). | Could |
| US-10 | As a player, I want a "no walls" mode where the snake comes out on the other side. | Could |
| US-11 | As a player, I want a pause key. | Could |

### Acceptance criteria (examples)

**US-3**
- Given the snake of length 3 and food right in front of the head, when the snake moves, then the length is 4, the score is +10, and the tail stays where it was.

**US-5**
- Given the head next to the wall, when the snake moves into the wall, then "Game over! Score: N" is shown.
- Moving into the cell where the **tail is right now** is **allowed** (the tail moves away in the same step) — unless the snake has just eaten.

**US-6**
- Given the snake moving up, when the player presses `S`, then the snake keeps moving up.

**US-7**
- Given a score that is in the top 5, the game asks for a name (1–12 characters) and saves the table to `scores.txt`.
- A broken or missing file must not crash the game.

### MVP scope

**MVP = US-1 … US-6** (turn-based). US-7 and US-8 are the release goal
if the velocity allows.

---

## Stage 2. Project Manager — Project Plan

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup + spike | — | repo, board; the TL tries `curses` on all team OSes and reports |
| 1 | "The snake moves" | US-1, US-2, US-6 | the snake walks on the field |
| 2 | "The snake eats and dies" | US-3, US-4, US-5 | **MVP** → Staging |
| 3 | "Hall of fame + real time" | US-7, US-8 (+ US-9…11) | **Release v1.0.0** |

A **spike** is a small experiment to answer a technical question before
planning. Its result decides if US-8 is realistic for this team.

### Risks

| Risk | Plan |
|------|------|
| `curses` is not installed on Windows | `pip install windows-curses`, or keep turn-based mode as the default |
| The game logic gets mixed with the input/output code and cannot be tested | TL: the logic is a pure function `step(state, direction) -> state` |
| The tail-collision rule is understood differently by developers | PO writes an exact acceptance criterion (see US-5) |

---

## Stage 3. Tech Lead — Technical Overview

### Architecture: logic separated from the screen

```
snake/
├── main.py         # menu, chooses the mode
├── game.py         # pure logic: state + step()
├── render.py       # state -> text
├── turn_mode.py    # input() loop
├── live_mode.py    # curses loop (US-8)
├── scores.py       # top-5 table
└── tests/
```

Both modes use **the same** `game.py`. Only the input loop is different.

### Data structures

```python
from collections import deque

state = {
    "width": 20, "height": 10,
    "snake": deque([(5, 7), (5, 6), (5, 5)]),   # head first, (row, col)
    "body": {(5, 7), (5, 6), (5, 5)},           # the same cells as a set
    "direction": (0, 1),                         # right
    "food": (2, 3),
    "score": 0,
    "alive": True,
}
```

Why both a `deque` and a `set`? The deque keeps the **order** (head and
tail in O(1)); the set answers "is this cell part of the snake?" in O(1).
With only a list, that check is O(n) — a good Chapter 15 discussion.

### The step function

```
step(state, new_direction):
    if new_direction is the opposite of state.direction: ignore it
    head = snake[0] + direction
    if head is outside the field: alive = False; return
    will_eat = (head == food)
    if not will_eat: remove the tail (from the deque and the set)
    if head in body: alive = False; return
    add the head
    if will_eat: score += 10; food = random free cell
```

Removing the tail **before** the self-collision check implements the
"moving into the tail is allowed" rule.

**Food:** build the list of free cells and use `random.choice`. If the
list is empty — the snake fills the field — the player wins.

### Screen

```
######################
#                    #
#   *                #
#        ooO         #
#                    #
######################
Score: 20     W/A/S/D + Enter, Q = quit
```

### Language notes

- **C++:** `std::deque<std::pair<int,int>>` + `std::set`; real time with `ncurses`.
- **JavaScript:** a browser version on `<canvas>` with `setInterval` is a natural real-time mode; the logic module stays the same.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| SNK-1 | Repo, structure, README skeleton | — | 1 | — | 0 |
| SNK-2 | Spike: `curses` on Windows/macOS/Linux, report to the team | US-8 | 2 | — | 0 |
| SNK-3 | `new_game(width, height, seed)` — initial state | US-1 | 2 | SNK-1 | 1 |
| SNK-4 | `render(state)` — walls, snake (`O` head, `o` body), food | US-1 | 2 | SNK-3 | 1 |
| SNK-5 | `step()` — movement only, ignore the reverse direction | US-2, US-6 | 3 | SNK-3 | 1 |
| SNK-6 | Turn-based loop: read `W A S D`/Enter/`Q` | US-2 | 2 | SNK-4, SNK-5 | 1 |
| SNK-7 | `step()` — eating and growing, score | US-3 | 3 | SNK-5 | 2 |
| SNK-8 | Food on a random free cell; full field = win | US-4 | 3 | SNK-7 | 2 |
| SNK-9 | Collisions with walls and the body (tail rule) | US-5 | 3 | SNK-5 | 2 |
| SNK-10 | "Game over" screen with the score | US-5 | 1 | SNK-9 | 2 |
| SNK-11 | Unit tests for `step()` | US-2…6 | 3 | SNK-9 | 2 |
| SNK-12 | Top-5 table in `scores.txt`, name input, broken file | US-7 | 3 | SNK-10 | 3 |
| SNK-13 | Real-time mode with `curses` | US-8 | 5 | SNK-2, SNK-10 | 3 |
| SNK-14 | Speed grows every 50 points | US-9 | 2 | SNK-13 | 3 |
| SNK-15 | README, release notes | — | 1 | SNK-10 | 3 |

**MVP total:** 26 SP (SNK-1 … SNK-11).

---

## Stage 5. Testing

### Unit tests

```python
from collections import deque
from game import step

UP, DOWN, LEFT, RIGHT = (-1, 0), (1, 0), (0, -1), (0, 1)

def make(snake, direction, food=(0, 0), w=10, h=10):
    return {"width": w, "height": h, "snake": deque(snake),
            "body": set(snake), "direction": direction,
            "food": food, "score": 0, "alive": True}

def test_moves_forward():
    s = make([(5, 5), (5, 4), (5, 3)], RIGHT, food=(0, 0))
    step(s, RIGHT)
    assert list(s["snake"]) == [(5, 6), (5, 5), (5, 4)]

def test_reverse_is_ignored():
    s = make([(5, 5), (5, 4)], RIGHT)
    step(s, LEFT)
    assert s["snake"][0] == (5, 6)

def test_eats_and_grows():
    s = make([(5, 5), (5, 4)], RIGHT, food=(5, 6))
    step(s, RIGHT)
    assert len(s["snake"]) == 3 and s["score"] == 10
    assert s["food"] not in s["body"]

def test_wall_kills():
    s = make([(5, 9), (5, 8)], RIGHT)
    step(s, RIGHT)
    assert not s["alive"]

def test_moving_into_tail_is_allowed():
    # a square: the head moves into the cell the tail is leaving
    s = make([(5, 5), (5, 6), (6, 6), (6, 5)], LEFT, food=(0, 0))
    step(s, DOWN)
    assert s["alive"]
```

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | Start | field with walls, snake of length 3, food visible |
| TC-2 | Press Enter several times | the snake moves straight |
| TC-3 | Moving right, press `A` | ignored, the snake continues right |
| TC-4 | Press `x`, `ww`, a space | ignored or "Unknown key", no crash |
| TC-5 | Eat food | +10, length +1, new food not on the snake |
| TC-6 | Drive into a wall | "Game over! Score: N" |
| TC-7 | Make a long snake and bite yourself | "Game over" |
| TC-8 | Get a top-5 score | name asked and saved; table printed |
| TC-9 | Real-time mode: hold no key | the snake keeps moving |
| TC-10 | Resize the terminal to be smaller than the field (real time) | a message "Make the window bigger", no crash |

---

## Stage 6. Staging

1. `v1.0.0-rc1` on `staging`; code freeze.
2. QA: fresh clone **on two different OSes** (this project has the
   highest platform risk); unit tests; TC-1 … TC-10.
3. **UAT:** the PO plays turn-based and real-time modes and checks the
   Must stories. If real time is unstable on one OS, the PO decides:
   release it as "experimental" or remove it from v1.0.0.

### Staging checklist

- [ ] fresh clone runs on at least two OSes
- [ ] all unit tests pass
- [ ] the README says which mode works on which OS
- [ ] no open critical / major bugs

---

## Stage 7. Production

1. PR `staging → main`, tag `v1.0.0`, GitHub Release.
2. **Leaderboard event:** students of other teams play; the best score
   gets into the team's README "Hall of Fame".
3. Hotfixes if needed → `v1.0.1`; retro.

### Release notes (example)

```
## Snake v1.0.0

New:
  - turn-based mode (all OSes)
  - real-time mode with curses (macOS, Linux; Windows needs `pip install windows-curses`)
  - top-5 high score table
Known issues:
  - real-time mode flickers in the VS Code terminal (minor)
How to run:
  python3 main.py
```

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Obstacles** | "Add some rocks inside the field!" — TL: is food spawning still correct? |
| **Food bug** | "The food appeared inside the snake!" — production bug → hotfix with a regression test. |
| **Two snakes** | "Can two people play on one keyboard?" — PO: big story, probably next release. |
| **Spike failed** | `curses` does not work for half of the team — PM and PO re-plan: US-8 moves to v1.1. |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** turn-based MVP works from a fresh clone;
`step()` is covered by unit tests, including the tail rule.
