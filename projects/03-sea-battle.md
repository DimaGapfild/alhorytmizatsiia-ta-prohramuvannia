# Project 3. Sea Battle

| | |
|---|---|
| **Type** | console game, player vs computer |
| **Difficulty** | ★★★★☆ |
| **Chapters used** | 5, 6, 8, 9 (matrices), 10 (functions, recursion), 12 (sets, tuples), 13 (errors), 14 (OOP) |
| **Ticket prefix** | `SEA` |
| **Team** | 5–6 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "Remember the paper game where you draw two 10×10 squares and shoot at
> each other's ships: 'B5!' — 'Miss!'? I want to play it against the
> computer. And the computer shouldn't be stupid — when it hits my ship,
> it should try to finish it, like a real person does."

| Question | Answer |
|----------|--------|
| Which ships? | The classic set: one 4-deck, two 3-deck, three 2-deck, four 1-deck. |
| Can ships touch? | No, not even by corners. |
| Do I place my ships myself? | Ideally yes, but automatic placement is OK for a start. |
| After a hit, do I shoot again? | Yes, a hit gives one more shot. |

---

## Stage 1. Product Owner — Product Description

### Vision

A console **Sea Battle** for one player against the computer. The
player sees two fields: their own fleet and the enemy's field with their
shots. The computer opponent plays **smart**: after a hit it hunts the
rest of the ship.

### Users

- **Casual player** — knows the paper game, wants a quick match.
- **Competitive player** — wants a computer that is hard to beat.

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a player, I want both fleets to be placed automatically by the rules, so that I can start playing at once. | Must |
| US-2 | As a player, I want to see my field and the enemy field side by side, so that I understand the game state. | Must |
| US-3 | As a player, I want to shoot by entering a cell like `E7`, so that I can attack. | Must |
| US-4 | As a player, I want to see "Miss", "Hit" or "Sunk!" after each shot, so that I know the result. | Must |
| US-5 | As a player, I want to shoot again after a hit, so that the rules match the paper game. | Must |
| US-6 | As a player, I want the computer to shoot back, so that it is a real match. | Must |
| US-7 | As a player, I want to win or lose when one fleet is fully sunk, and see the final statistics. | Must |
| US-8 | As a player, I want wrong input and repeated shots to be refused without losing my turn. | Must |
| US-9 | As a competitive player, I want the computer to finish a ship after it hits it ("hunt mode"). | Should |
| US-10 | As a player, I want to place my ships myself, so that I can use my own strategy. | Should |
| US-11 | As a player, I want the cells around a sunk ship to be marked automatically, so that I do not waste shots. | Should |
| US-12 | As a player, I want to save the game and continue later. | Could |
| US-13 | As two people, we want to play on one computer against each other. | Won't (this release) |

### Acceptance criteria (examples)

**US-1**
- The fleet always has exactly 10 ships: 1×4, 2×3, 3×2, 4×1.
- No two ships touch each other, not even diagonally.
- Ships are horizontal or vertical and stay inside the 10×10 field.

**US-4**
- Given the last unhit cell of a ship, when I hit it, then I see "Sunk!", not "Hit".

**US-8**
- Given a cell I already shot, when I enter it again, then I see "You already shot there" and still have my turn.

**US-9**
- Given the computer hit a ship that is not sunk, then its next shots go to the neighbouring cells (up, down, left, right) until the ship is sunk.
- After two hits in a line, it shoots only along that line.

### MVP scope

**MVP = US-1 … US-8** (a random computer opponent). The smart opponent
(US-9) is the most valuable Should story — the PO asks to plan it right
after the MVP.

---

## Stage 2. Project Manager — Project Plan

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup | — | repo, board, roles, TL's interface contract agreed |
| 1 | "The fleet exists" | US-1, US-2 | random valid fleets are shown on screen |
| 2 | "We can fight" | US-3, US-4, US-5, US-6, US-7, US-8 | **MVP** → Staging |
| 3 | "The computer is smart" | US-9, US-11, US-10 | **Release v1.0.0** |

The PM notes that Sprint 2 is the heaviest: it is planned with the
velocity measured in Sprint 1, and US-7 is the first candidate to move
if the sprint is overloaded.

### Risks

| Risk | Plan |
|------|------|
| Random placement can loop forever (no space left for a ship) | TL: restart the whole placement after N failed tries |
| Off-by-one errors in coordinates (`J10`, row 0 vs 1) | parser tests for every border cell |
| The AI is the hardest part and nobody can review it | two developers pair on it; TL reviews |
| Bugs that depend on random | every random function takes a `seed` |

---

## Stage 3. Tech Lead — Technical Overview

### Classes (Chapter 14)

```
Ship        cells: set[tuple[int,int]], hits: set[...]
            is_sunk() -> bool

Board       size=10, ships: list[Ship], shots: set[...]
            can_place(cells) -> bool
            place(ship)
            place_fleet_randomly(rng)
            receive_shot(cell) -> "miss" | "hit" | "sunk" | "repeat"
            all_sunk() -> bool
            neighbours(cell) -> set[...]       # 8 cells around

Player      name, board (own), enemy_view
            choose_shot() -> cell              # abstract

HumanPlayer(Player)     reads and parses input
ComputerPlayer(Player)  random or hunt mode

Game        two players, whose turn, main loop, statistics
```

```
sea_battle/
├── main.py
├── ship.py
├── board.py
├── players.py
├── render.py        # prints two fields side by side
├── coords.py        # "E7" <-> (6, 4)
└── tests/
```

### Key algorithms

**Random placement**

```
for length in [4, 3, 3, 2, 2, 2, 1, 1, 1, 1]:
    repeat:
        choose a random direction and a random start cell
        build the list of cells
    until all cells are inside AND none of them or their neighbours is taken
```

Keep a set of "forbidden" cells: every ship cell plus its 8 neighbours.
Checking a cell is then `O(1)` (Chapter 12, Chapter 15).

**Smart computer (hunt / target)**

```
mode = "hunt"
hunt:   shoot a random cell not shot before
        (better: only cells of one colour of a chessboard — every ship ≥ 2 is found faster)
        if hit → mode = "target", remember the hit
target: candidates = untouched neighbours (up/down/left/right) of the hits
        if two hits in a line → only cells continuing that line
        if sunk → mark the ring around the ship as "shot", mode = "hunt"
```

### Screen

```
     YOUR FLEET                ENEMY
   A B C D E F G H I J     A B C D E F G H I J
 1 . # # # # . . . . .   1 . . . . . . . . . .
 2 . . . . . . . X . .   2 . o . . . . . . . .
 3 # . . o . . . # . .   3 . . . X X . . . . .
...
 # ship   X hit   o miss   . unknown / water
Your shot:
```

### Code rules

- The enemy fleet is **never** printed — the render function gets only the
  "view" (shots and results), not the ships. This prevents a cheat bug.
- Coordinates inside the program are always `(row, col)` from 0.
  Letters and numbers exist only in `coords.py`.

### Language notes

- **C++:** `std::set<std::pair<int,int>>`, a base class `Player` with a
  virtual `chooseShot()`.
- **JavaScript:** sets of strings like `"6,4"` because JS sets compare
  arrays by reference.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| SEA-1 | Repo, structure, class skeletons agreed with TL | — | 2 | — | 0 |
| SEA-2 | `coords.py`: parse `A1`…`J10`, format back, errors | US-3, US-8 | 2 | SEA-1 | 1 |
| SEA-3 | `Ship` class with `is_sunk()` | US-1 | 1 | SEA-1 | 1 |
| SEA-4 | `Board.can_place`, `Board.neighbours`, `Board.place` | US-1 | 3 | SEA-3 | 1 |
| SEA-5 | `Board.place_fleet_randomly(rng)` with restart on failure | US-1 | 5 | SEA-4 | 1 |
| SEA-6 | `render.py`: two fields side by side, legend | US-2 | 3 | SEA-4 | 1 |
| SEA-7 | `Board.receive_shot` → miss / hit / sunk / repeat | US-4, US-8 | 3 | SEA-4 | 2 |
| SEA-8 | `HumanPlayer.choose_shot` with input validation | US-3, US-8 | 2 | SEA-2 | 2 |
| SEA-9 | `ComputerPlayer` — random shots, never repeat | US-6 | 2 | SEA-7 | 2 |
| SEA-10 | `Game` loop: turns, extra shot after a hit, end of game | US-5, US-6, US-7 | 5 | SEA-7, SEA-8, SEA-9 | 2 |
| SEA-11 | Final statistics: shots, hits, accuracy % | US-7 | 1 | SEA-10 | 2 |
| SEA-12 | Unit tests: placement rules, shots, sinking | US-1, US-4 | 3 | SEA-5, SEA-7 | 2 |
| SEA-13 | Hunt / target AI | US-9 | 8 | SEA-10 | 3 |
| SEA-14 | Auto-mark the ring around a sunk ship (both sides) | US-11 | 2 | SEA-7 | 3 |
| SEA-15 | Manual placement: `A1 h` / `C3 v`, with validation | US-10 | 5 | SEA-4 | 3 |
| SEA-16 | README + release notes | — | 1 | SEA-10 | 3 |

**MVP total:** 32 SP (SEA-1 … SEA-12).

---

## Stage 5. Testing

### Unit tests

```python
import random
from board import Board
from ship import Ship

def test_fleet_composition():
    b = Board()
    b.place_fleet_randomly(random.Random(42))
    lengths = sorted(len(s.cells) for s in b.ships)
    assert lengths == [1, 1, 1, 1, 2, 2, 2, 3, 3, 4]

def test_ships_do_not_touch():
    for seed in range(200):              # many random fleets
        b = Board()
        b.place_fleet_randomly(random.Random(seed))
        for s in b.ships:
            for other in b.ships:
                if s is not other:
                    ring = set().union(*(b.neighbours(c) for c in s.cells))
                    assert not (ring & other.cells), f"seed {seed}"

def test_shot_results():
    b = Board()
    b.place(Ship({(0, 0), (0, 1)}))
    assert b.receive_shot((5, 5)) == "miss"
    assert b.receive_shot((0, 0)) == "hit"
    assert b.receive_shot((0, 0)) == "repeat"
    assert b.receive_shot((0, 1)) == "sunk"
    assert b.all_sunk()
```

**AI test:** play 100 games "computer vs a fixed fleet" with different
seeds and compare the average number of shots: random ≈ 95, hunt/target
must be clearly lower (≈ 60–65). Put the numbers into the test report.

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | Start | two fields; your fleet visible, the enemy field empty |
| TC-2 | Shoot `J10` and `A1` | both accepted (border cells) |
| TC-3 | Shoot `K1`, `A11`, `A`, `10A` | "Invalid cell", turn kept |
| TC-4 | Shoot the same cell twice | "You already shot there", turn kept |
| TC-5 | Hit a ship | "Hit", you shoot again |
| TC-6 | Hit the last cell of a ship | "Sunk!", the ring around is marked (US-11) |
| TC-7 | Miss | the computer shoots; its result is printed |
| TC-8 | Sink the whole enemy fleet | "You won!" + statistics |
| TC-9 | Let the computer win | "You lost" + statistics; the enemy fleet is revealed |
| TC-10 | Watch the computer after it hits | it shoots next to the hit (US-9) |

---

## Stage 6. Staging

1. `staging` tagged `v1.0.0-rc1`; code freeze.
2. QA: fresh clone, unit tests, TC-1 … TC-10, and the AI statistics run.
3. **UAT:** the PO plays at least **three full games** and checks
   every Must story. The PO also plays one game in "debug mode"
   (`python3 main.py --reveal`) to check that the enemy fleet follows the
   placement rules.
4. The PM reviews the board: every Done ticket really has a merged PR.

### Staging checklist

- [ ] fresh clone starts with the README command
- [ ] all unit tests pass, including the 200-seed placement test
- [ ] the enemy fleet is never shown during a normal game
- [ ] no open critical / major bugs
- [ ] release notes drafted

---

## Stage 7. Production

1. PR `staging → main`, approvals from the TL and the PO; tag `v1.0.0`;
   GitHub Release.
2. **Tournament:** every team plays against the computer of another
   team. The team whose AI wins most often gets a bonus.
3. Bugs from the tournament → bug reports → `hotfix/...` → `v1.0.1`.
4. Retro.

### Release notes (example)

```
## Sea Battle v1.0.0

New:
  - player vs computer on a 10×10 field, classic fleet
  - smart computer: finishes damaged ships
  - the ring around a sunk ship is marked automatically
Known issues:
  - manual ship placement is not included (planned for v1.1)
How to run:
  python3 main.py            normal game
  python3 main.py --reveal   show the enemy fleet (for testing)
```

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Cheater found** | "My friend says the enemy ships are visible if you scroll up!" — critical production bug → hotfix. |
| **Difficulty levels** | "The computer is now too strong for my kid. Add 'easy'." — PO: new story; TL: strategy pattern (`RandomAI` / `SmartAI`). |
| **Another fleet** | "In my country we play with a 5-deck ship." — TL checks if the fleet is a constant in one place. |
| **Sick day** | the developer of the AI is absent for the sprint — PM decides: pair-switch or cut US-9 from the release. |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** MVP with a random opponent works from a fresh
clone; placement rules are covered by unit tests.
