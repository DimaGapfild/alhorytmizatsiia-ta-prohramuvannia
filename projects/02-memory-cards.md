# Project 2. Memory — Find the Pair

| | |
|---|---|
| **Type** | console game |
| **Difficulty** | ★★☆☆☆ (good first project for the flow) |
| **Chapters used** | 5 (branching), 6 (loops), 8 (lists), 9 (matrices), 10 (functions), 13 (files, errors) |
| **Ticket prefix** | `MEM` |
| **Team** | 4–5 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "My little brother plays a card game: all cards lie face down, you
> turn over two, and if they are the same — you keep them open. If not —
> you turn them back and try to remember where they were. I want this
> game on the computer, so he can play it alone. And I want to see if he
> gets better — maybe count how many tries it took."

**Questions the team should ask the Client** (and the Client's answers):

| Question | Answer |
|----------|--------|
| Pictures or letters on the cards? | Letters or symbols are fine for now. |
| How big is the board? | Small for a start. Bigger later would be nice. |
| One player or two? | One. Two players is a "maybe later". |
| Should the results be remembered after the game is closed? | Yes, the best result. |

---

## Stage 1. Product Owner — Product Description

### Vision

A console game for **one player** that trains memory. The player opens
cards two at a time and tries to find all pairs in **as few moves as
possible**. The best result is saved, so the player can try to beat it.

### Users

- **Kid player (7–12 years)** — needs simple input and clear messages.
- **Parent** — wants to see progress (the best score).

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a player, I want to see the board with closed cards and coordinates, so that I know which cards I can choose. | Must |
| US-2 | As a player, I want to open two cards by entering their coordinates, so that I can look for a pair. | Must |
| US-3 | As a player, I want matched cards to stay open and unmatched cards to close again, so that the game follows the real rules. | Must |
| US-4 | As a player, I want to see the number of moves, so that I know how well I play. | Must |
| US-5 | As a player, I want a clear message when I enter a wrong cell, so that the game does not crash. | Must |
| US-6 | As a player, I want to see a "You won!" message with my result, so that I know the game is over. | Must |
| US-7 | As a parent, I want the best result to be saved between games, so that I can see progress. | Should |
| US-8 | As a player, I want to choose the size of the board (easy 4×4, hard 6×6), so that the game stays interesting. | Should |
| US-9 | As a player, I want to play again without restarting the program. | Could |
| US-10 | As two players, we want to take turns and count our pairs. | Won't (this release) |

### Acceptance criteria (examples)

**US-2**
- Given a board 4×4, when I enter `B3`, then the card in column B, row 3 opens.
- Given an opened card `B3`, when I enter `B3` again as the second card, then I see "This card is already open" and choose again.
- Input is not case-sensitive: `b3` = `B3`.

**US-3**
- Given two open cards with the same symbol, then they stay open until the end.
- Given two open cards with different symbols, then the board is shown with both cards for the player to see, the player presses Enter, the screen is cleared and both cards are closed again.

**US-7**
- Given a finished game with 14 moves and a saved best of 18, then the new best is 14 and the message "New record!" is shown.
- Given that the file with the best result is missing or broken, the game still starts (the best result is "none yet").

### MVP scope

**MVP = US-1 … US-6** (one board size, 4×4). US-7 and US-8 if the team
has capacity; US-9 is a bonus.

**Out of scope:** graphics, sounds, a timer, network play.

---

## Stage 2. Project Manager — Project Plan

### Sprints

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup | — | repo, branches `main`/`staging`, board, README skeleton, roles fixed |
| 1 | "I can open cards" | US-1, US-2, US-5 | the board is shown and cards can be opened |
| 2 | "I can win" | US-3, US-4, US-6 | **MVP complete** → Staging |
| 3 | "I want to beat my record" | US-7, US-8 (+ US-9) | **Release v1.0.0** |

### Roles (example for 5 students)

| Student | Role | Also develops |
|---------|------|---------------|
| A | PO | US-7 (best score file) |
| B | PM | input parsing |
| C | TL | board module, reviews |
| D | Dev | game loop |
| E | QA | test cases, unit tests |

### Risks

| Risk | Probability | Impact | Plan |
|------|-------------|--------|------|
| "Clearing the screen" works differently on Windows / macOS / Linux | high | medium | TL chooses one approach (printing empty lines) in Sprint 0 |
| Two developers change the same file → merge conflicts | medium | medium | TL splits the code into modules; small PRs |
| Random board makes bugs hard to repeat | high | medium | the board generator takes a `seed` parameter for tests |

### Meetings

- **Planning** at the start of each sprint — choose tickets, check estimates.
- **Stand-up** at the start of every lesson — each member answers: what I did, what I will do, what blocks me.
- **Demo** at the end of each sprint — show the working program to the PO.
- **Retro** at the end of the project.

---

## Stage 3. Tech Lead — Technical Overview

### Modules

```
memory/
├── main.py         # entry point: menu, game loop
├── board.py        # creating, shuffling and printing the board
├── input_parser.py # "B3" → (row, col), validation
├── scores.py       # load/save the best result (best_score.txt)
├── tests/
│   ├── test_board.py
│   └── test_input_parser.py
└── README.md
```

The **game logic does not print** and **does not read input** — only
`main.py` talks to the user. This makes the logic easy to test.

### Data structures

```python
# The board: a matrix (Chapter 9) of symbols
cards = [["A", "C", "B", "D"],
         ["D", "A", "C", "B"],
         ...]

# Which cards are open: a matrix of booleans of the same size
opened = [[False] * cols for _ in range(rows)]
```

### Key algorithms

- **Creating the board:** take `rows * cols // 2` symbols, double the list,
  shuffle it with `random.shuffle` (or Fisher–Yates by hand — Chapter 8),
  then cut it into rows.
- **Parsing input:** letter → column index (`ord(letter) - ord("A")`),
  number → row index (`int(digits) - 1`); check the range; raise
  `ValueError` with a clear message.
- **Win check:** all values in `opened` are `True`.

### Function signatures (agreed contract)

```python
def create_board(rows: int, cols: int, seed: int | None = None) -> list[list[str]]: ...
def render(cards, opened, reveal: set[tuple[int, int]] = set()) -> str: ...
def parse_cell(text: str, rows: int, cols: int) -> tuple[int, int]: ...
def is_won(opened) -> bool: ...
def load_best(path: str) -> int | None: ...
def save_best(path: str, moves: int) -> None: ...
```

### Screen

```
    A B C D
 1  * * * *
 2  * A * *
 3  * * * A
 4  * * * *
Moves: 5   Best: 14
Choose the first card:
```

### Code rules

- Names in `snake_case`, constants in `UPPER_CASE`.
- Every function has a docstring.
- No global variables except constants.
- `rows * cols` must be even — check it in `create_board`.

### Language notes

- **C++:** `std::vector<std::vector<char>>`, `std::shuffle` with `std::mt19937`.
- **JavaScript:** Node.js with `readline`; shuffle with Fisher–Yates.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| MEM-1 | Repo, branches, README skeleton, folder structure | — | 1 | — | 0 |
| MEM-2 | `create_board(rows, cols, seed)` — pairs, shuffle, even-size check | US-1 | 3 | MEM-1 | 1 |
| MEM-3 | `render()` — coordinates header, `*` for closed cards | US-1 | 2 | MEM-2 | 1 |
| MEM-4 | `parse_cell()` — "B3"/"b3" → (2, 1), errors for bad input | US-2, US-5 | 3 | MEM-1 | 1 |
| MEM-5 | Game loop: ask for two cards, refuse an already open card | US-2 | 3 | MEM-3, MEM-4 | 1 |
| MEM-6 | Match logic: keep pairs open, close others after Enter, clear screen | US-3 | 3 | MEM-5 | 2 |
| MEM-7 | Move counter, shown under the board | US-4 | 1 | MEM-5 | 2 |
| MEM-8 | `is_won()` + "You won in N moves!" | US-6 | 2 | MEM-6 | 2 |
| MEM-9 | Unit tests for board and parser | US-1, US-5 | 3 | MEM-2, MEM-4 | 2 |
| MEM-10 | `load_best` / `save_best`, handle missing or broken file | US-7 | 3 | MEM-8 | 3 |
| MEM-11 | Difficulty menu: 4×4 or 6×6 | US-8 | 2 | MEM-8 | 3 |
| MEM-12 | "Play again? (y/n)" | US-9 | 1 | MEM-8 | 3 |
| MEM-13 | README: how to run, how to play, screenshots of the console | — | 1 | MEM-8 | 3 |

**MVP total:** 21 SP (MEM-1 … MEM-9).

---

## Stage 5. Testing

### Unit tests (QA + Devs)

```python
from board import create_board, is_won
from input_parser import parse_cell

def test_board_has_pairs():
    cards = create_board(4, 4, seed=1)
    flat = [c for row in cards for c in row]
    for symbol in set(flat):
        assert flat.count(symbol) == 2

def test_board_odd_size_is_error():
    try:
        create_board(3, 3)
        assert False, "expected ValueError"
    except ValueError:
        pass

def test_same_seed_same_board():
    assert create_board(4, 4, seed=7) == create_board(4, 4, seed=7)

def test_parse_cell():
    assert parse_cell("B3", 4, 4) == (2, 1)
    assert parse_cell(" b3 ", 4, 4) == (2, 1)

def test_parse_cell_out_of_range():
    for bad in ["E1", "A5", "A0", "", "33", "BB", "B"]:
        try:
            parse_cell(bad, 4, 4)
            assert False, f"{bad!r} must be rejected"
        except ValueError:
            pass
```

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | Start the game | a 4×4 board of `*`, coordinates A–D and 1–4, "Moves: 0" |
| TC-2 | Enter `A1`, then `A1` | "This card is already open", asks again |
| TC-3 | Enter `Z9` | "Invalid cell", asks again, no crash |
| TC-4 | Open two different cards, press Enter | the screen is cleared, both cards closed, moves +1 |
| TC-5 | Open a pair | both stay open; moves +1 |
| TC-6 | Open a card that is already part of a found pair | "This card is already open" |
| TC-7 | Find all pairs | "You won in N moves!" |
| TC-8 | Delete `best_score.txt`, finish a game | the file is created with the result |
| TC-9 | Write `abc` into `best_score.txt`, start the game | the game starts, best = "none yet" |
| TC-10 | Press Ctrl+C during the game | the program exits with "Bye!" and no traceback |

### Test report (end of each sprint)

```
Sprint 2 test report
Test cases: 10 run, 8 passed, 2 failed
Bugs: BUG-1 (major) — fixed; BUG-2 (minor) — open
Recommendation: ready for staging after BUG-1 is merged
```

---

## Stage 6. Staging

1. The TL merges all MVP tickets into `staging` and tags `v1.0.0-rc1`.
2. **Code freeze:** from now on only bug fixes go into `staging`.
3. QA makes a **fresh clone** into a new folder and follows only the README.
4. QA runs all test cases TC-1 … TC-10 and the unit tests.
5. **UAT (User Acceptance Test):** the PO plays two full games — one on
   4×4, one on 6×6 — and checks every Must story against its acceptance
   criteria.
6. Bugs found → bug reports → fixes → `v1.0.0-rc2` → repeat.

### Staging checklist

- [ ] the program starts from a fresh clone with the README command
- [ ] all unit tests pass
- [ ] all Must stories accepted by the PO
- [ ] no open critical / major bugs
- [ ] release notes drafted

**Exit:** the PO writes in the PR "staging → main": *"Accepted for release."*

---

## Stage 7. Production

1. PR `staging → main`, approved by the TL and the PO.
2. Tag `v1.0.0` on `main`; create a GitHub Release with the release notes.
3. **Release demo:** another team clones the repo, plays, and gives
   feedback (this is your "real user").

### Release notes (example)

```
## Memory v1.0.0

New:
  - 4×4 and 6×6 boards
  - move counter and best result saved in best_score.txt
  - clear messages for wrong input
Known issues:
  - BUG-2 (minor): on Windows the screen is not fully cleared on very tall terminals
How to run:
  python3 main.py
```

### After the release

- Feedback from the other team goes into the backlog as new stories.
- If they find a crash → `hotfix/MEM-14-...` → `v1.0.1`.
- **Retro** — see the template in [the rules](00-dev-flow.md#6-templates).

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Emoji** | "My brother can't read letters well. Can the cards be animals? 🐶🐱🐭" — check that the board is still aligned (emoji are wider). |
| **Cheat** | "He found out he can type the same card twice and see it for free!" — a production bug → hotfix. |
| **Two players** | "His friend came — can they play together?" — PO decides: backlog or replace US-9. |
| **Timer** | "Moves are not enough, I want time too." — new story; TL checks `time.monotonic()`. |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** MVP (US-1 … US-6) works from a fresh clone,
there are stories and tickets, and at least 5 unit tests pass.
