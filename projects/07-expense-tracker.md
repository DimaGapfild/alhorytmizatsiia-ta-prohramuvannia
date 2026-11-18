# Project 7. Personal Budget (Expense Tracker)

| | |
|---|---|
| **Type** | console application |
| **Difficulty** | ★★★☆☆ |
| **Chapters used** | 4 (arithmetic), 5, 7 (strings), 10, 11 (sorting), 12 (dictionaries), 13 (CSV files, errors), 14 (OOP) |
| **Ticket prefix** | `EXP` |
| **Team** | 4–5 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "I'm a student and my money always disappears by the middle of the
> month. I want to write down what I spend — food, transport, coffee —
> and see where the money goes. And please warn me before I spend too much!"

| Question | Answer |
|----------|--------|
| Only expenses, or income too? | Both, but expenses are more important. |
| Which currency? | One currency, hryvnia. |
| Categories — fixed or my own? | A default list, but I want to add my own. |
| Can I open it in Excel? | Yes, that would be great — CSV is fine. |

---

## Stage 1. Product Owner — Product Description

### Vision

A console tool that makes the user's money **visible**: quick input of
expenses and income, a monthly report by categories, and a warning when
a category budget is close to its limit.

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a user, I want to add an expense: amount, category, optional comment; the date is today by default. | Must |
| US-2 | As a user, I want to add an income. | Must |
| US-3 | As a user, I want to see the list of operations for a month, newest first. | Must |
| US-4 | As a user, I want to see my balance (income − expenses) for a month. | Must |
| US-5 | As a user, I want a report: how much I spent per category, with percentages. | Must |
| US-6 | As a user, I want my data saved in a CSV file that opens in Excel. | Must |
| US-7 | As a user, I want wrong amounts (`abc`, `-5`, `0`) to be refused with a clear message. | Must |
| US-8 | As a user, I want to delete a wrong operation. | Must |
| US-9 | As a user, I want to set a monthly limit per category and get a warning at 80 % and 100 %. | Should |
| US-10 | As a user, I want to add my own categories. | Should |
| US-11 | As a user, I want a text bar chart of categories in the report. | Should |
| US-12 | As a user, I want to compare this month with the previous one. | Could |
| US-13 | As a user, I want to import operations from my bank's CSV. | Won't (this release) |

### Acceptance criteria (examples)

**US-1**
- `spend 85.50 food lunch` → expense 85.50, category food, comment "lunch", today's date.
- `spend 85,50 food` — a comma is also accepted as the decimal separator.
- Unknown category → "Unknown category 'fod'. Did you mean 'food'?" (Could: a suggestion).

**US-5** — the report format:

```
November 2026
Category     Spent      %
food       2 340.00   46.8 %  ██████████████
transport    980.00   19.6 %  ██████
coffee       640.00   12.8 %  ████
other      1 040.00   20.8 %  ██████
TOTAL      5 000.00  100.0 %
Income     6 000.00     Balance  +1 000.00
```

**US-9**
- Given a food limit of 3 000 and 2 300 already spent, when I add 200 food, then I see "⚠ food: 2 500 of 3 000 (83 %)".
- When the total goes over the limit — "⛔ food limit exceeded by 150.00".

**Money rule:** amounts are stored as **integers in kopecks** (85.50 →
8550). Floats lose cents: `0.1 + 0.2 != 0.3` (Chapter 4). This is a TL
decision that the PO must know about, because it changes the CSV format.

### MVP scope

**MVP = US-1 … US-8.** Release goal: + US-9, US-10, US-11.

---

## Stage 2. Project Manager — Project Plan

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup | — | repo, board, the money rule and the CSV format agreed |
| 1 | "Write it down" | US-1, US-2, US-6, US-7 | operations are saved in CSV |
| 2 | "Where did it go?" | US-3, US-4, US-5, US-8 | **MVP** → Staging |
| 3 | "Stop me!" | US-9, US-10, US-11 (+ US-12) | **Release v1.0.0** |

### Risks

| Risk | Plan |
|------|------|
| Float rounding errors in money | integers in kopecks, a single `parse_money` / `format_money` pair |
| Excel opens CSV with wrong columns (`;` vs `,`) or wrong letters | TL tests the file in Excel / LibreOffice in Sprint 1; `utf-8-sig` for Excel |
| The month report has edge cases (first/last day, empty month) | QA writes tests for them before the code |

---

## Stage 3. Tech Lead — Technical Overview

### Structure

```
budget/
├── main.py          # REPL and commands
├── money.py         # parse_money("85,50") -> 8550, format_money(8550) -> "85.50"
├── models.py        # Operation, Budget (operations, categories, limits)
├── reports.py       # month list, category totals, bars, comparison
├── storage.py       # operations.csv, settings.json (categories, limits)
└── tests/
```

### Data model

```python
class Operation:
    id: int
    date: date
    kind: str          # "expense" | "income"
    amount: int        # kopecks, always > 0
    category: str
    comment: str

class Budget:
    operations: list[Operation]
    categories: set[str]             # default: food, transport, coffee, home, fun, other
    limits: dict[str, int]           # category → kopecks per month
    add(...) -> Operation
    remove(id)
    month(year, month) -> list[Operation]
```

`operations.csv`:

```
id,date,kind,amount,category,comment
1,2026-11-13,expense,8550,food,lunch
2,2026-11-13,income,600000,salary,
```

Use the `csv` module — it handles commas and quotes in comments.

### Key algorithms

- **Category totals** (Chapter 12): one pass with a `dict` —
  `totals[op.category] = totals.get(op.category, 0) + op.amount`.
- **Sorting the report** (Chapter 11): by amount, descending; "other" always last.
- **Bars:** `bar_len = round(amount / max_amount * 20)`.
- **Percentages:** computed from integers; the sum of rounded percentages
  may be 99.9 % or 100.1 % — the PO must accept this or ask for a fix.
- **Limit check:** after every `spend`, compute the month total for the
  category and compare with 80 % and 100 % of the limit.

### Commands

```
spend <amount> <category> [comment]     earn <amount> [source]
list [YYYY-MM]          report [YYYY-MM]        delete <id>
limit <category> <amount>                category add <name>
compare                  help                    quit
```

### Language notes

- **C++:** `long long` for kopecks; `std::map<std::string, long long>` for totals.
- **JavaScript:** `Number` is a float — still store integers in kopecks;
  `toLocaleString("uk-UA")` for formatting.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| EXP-1 | Repo, structure, money rule + CSV format in README | — | 1 | — | 0 |
| EXP-2 | `money.py`: parse (`.`/`,`, errors) and format | US-1, US-7 | 3 | EXP-1 | 1 |
| EXP-3 | `Operation`, `Budget.add`, default categories | US-1, US-2 | 2 | EXP-1 | 1 |
| EXP-4 | `spend` / `earn` commands with validation | US-1, US-2, US-7 | 3 | EXP-2, EXP-3 | 1 |
| EXP-5 | `storage.py`: CSV load/save, broken lines skipped with a warning | US-6 | 3 | EXP-3 | 1 |
| EXP-6 | `list [month]`, newest first | US-3 | 2 | EXP-3 | 2 |
| EXP-7 | Balance for a month | US-4 | 1 | EXP-6 | 2 |
| EXP-8 | Category report with percentages | US-5 | 3 | EXP-6 | 2 |
| EXP-9 | `delete <id>` | US-8 | 1 | EXP-5 | 2 |
| EXP-10 | Unit tests: money, totals, month filter, CSV | US-1…8 | 3 | EXP-5, EXP-8 | 2 |
| EXP-11 | Limits in `settings.json`, warnings at 80 % / 100 % | US-9 | 3 | EXP-8 | 3 |
| EXP-12 | Custom categories | US-10 | 2 | EXP-3 | 3 |
| EXP-13 | Text bars in the report | US-11 | 1 | EXP-8 | 3 |
| EXP-14 | Compare with the previous month | US-12 | 2 | EXP-8 | 3 |
| EXP-15 | README, release notes | — | 1 | EXP-10 | 3 |

**MVP total:** 22 SP (EXP-1 … EXP-10).

---

## Stage 5. Testing

### Unit tests

```python
from datetime import date
from money import parse_money, format_money
from models import Budget
from reports import category_totals

def test_parse_money():
    assert parse_money("85.50") == 8550
    assert parse_money("85,5") == 8550
    assert parse_money("100") == 10000
    for bad in ["abc", "-5", "0", "1.234", ""]:
        try:
            parse_money(bad)
            assert False, bad
        except ValueError:
            pass

def test_no_float_errors():
    b = Budget()
    for _ in range(10):
        b.add("expense", parse_money("0.10"), "coffee", day=date(2026, 11, 1))
    assert category_totals(b.month(2026, 11)) == {"coffee": 100}   # exactly 1.00

def test_month_borders():
    b = Budget()
    b.add("expense", 100, "food", day=date(2026, 10, 31))
    b.add("expense", 200, "food", day=date(2026, 11, 1))
    b.add("expense", 300, "food", day=date(2026, 11, 30))
    assert len(b.month(2026, 11)) == 2

def test_format_money():
    assert format_money(234000) == "2 340.00"
    assert format_money(5) == "0.05"
```

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | `spend 85.50 food lunch`, `list` | the operation is shown with today's date |
| TC-2 | `spend abc food`, `spend -5 food`, `spend 0 food` | three clear errors |
| TC-3 | `spend 10 fod` | "Unknown category" |
| TC-4 | `earn 6000`, several `spend`, `report` | totals, percentages, balance correct (check with a calculator) |
| TC-5 | `report 2025-01` (no data) | "No operations in January 2025" |
| TC-6 | Comment with a comma: `spend 50 fun movie, popcorn` | saved and loaded correctly |
| TC-7 | Open `operations.csv` in Excel / LibreOffice | columns are correct; Cyrillic is readable |
| TC-8 | Break one line in the CSV by hand, start | warning "line 5 skipped", the rest is loaded |
| TC-9 | `limit food 3000`, spend up to 85 % | warning ⚠ |
| TC-10 | Spend over the limit | warning ⛔ with the exceeded amount |
| TC-11 | `delete 99` | "No operation with ID 99" |

---

## Stage 6. Staging

1. `v1.0.0-rc1`; code freeze; fresh clone.
2. QA: unit tests, TC-1 … TC-11.
3. **UAT:** the PO enters a **realistic month** of student spending
   (≈ 40 operations) from a prepared list and compares the app's report
   with a spreadsheet calculation. The numbers must match **to the
   kopeck**.
4. The PM checks that the README explains the CSV columns.

### Staging checklist

- [ ] fresh clone works
- [ ] all unit tests pass, including the float test
- [ ] the report matches the spreadsheet to the kopeck
- [ ] the CSV opens correctly in a spreadsheet program
- [ ] no open critical / major bugs

---

## Stage 7. Production

1. PR `staging → main`, tag `v1.0.0`, GitHub Release.
2. Another team tracks their (pretend) spending with your app and
   reports bugs.
3. Hotfix if needed → `v1.0.1`. Retro.

### Release notes (example)

```
## Budget v1.0.0

New:
  - expenses and income, categories, comments
  - monthly list, balance and category report with bars
  - category limits with warnings at 80 % and 100 %
  - data in operations.csv (opens in Excel / LibreOffice)
Known issues:
  - percentages may add up to 99.9 % because of rounding
How to run:
  python3 main.py
```

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Second currency** | "I got 50 euros from my aunt!" — PO: new story or Won't? TL: what breaks if `currency` is added? |
| **Wrong total** | "The report says 99.99 but I spent 100!" — a production bug; QA looks for a float somewhere → hotfix. |
| **Regular payments** | "My phone bill is the same every month — can it be added automatically?" — PO: backlog. |
| **Privacy** | "My roommate can read my CSV!" — the team discusses: is this in scope? (a good "Won't" with a reason). |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** MVP works from a fresh clone; money is stored
without float errors; the report is correct for a test month.
