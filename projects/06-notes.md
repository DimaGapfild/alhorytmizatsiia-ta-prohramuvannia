# Project 6. Notes with Tags and Search

| | |
|---|---|
| **Type** | console application |
| **Difficulty** | ★★★☆☆ |
| **Chapters used** | 7 (strings), 8, 10, 11 (sorting, search), 12 (dictionaries, sets), 13 (files, errors), 14 (OOP), 15 (complexity) |
| **Ticket prefix** | `NOTE` |
| **Team** | 4–6 students |

Read [the rules of the game](00-dev-flow.md) first.

---

## Stage 0. The Idea (from the Client)

> "I write lecture notes, ideas and recipes in random text files, and
> then I can never find anything. I want one place for all my notes, where
> I can put tags like `#python` or `#exam` and search: 'show me all notes
> about recursion'. And I want to get them out as normal files when I need to."

| Question | Answer |
|----------|--------|
| How long are the notes? | From one line to a few pages. |
| How do I write a long note in a console? | Line by line; an empty line with only `.` ends the note. |
| Search — the exact word or a part of it? | A part is fine. And it must not care about capital letters. |
| Export to which format? | Markdown or plain text. |

---

## Stage 1. Product Owner — Product Description

### Vision

A personal **notebook in the terminal**: create, read, edit and delete
notes, organise them with tags, and find any note in a second.

### User stories

| ID | Story | Priority |
|----|-------|----------|
| US-1 | As a user, I want to create a note with a title and a multi-line text. | Must |
| US-2 | As a user, I want to see the list of notes (ID, title, tags, date) — newest first. | Must |
| US-3 | As a user, I want to open a note and read the full text. | Must |
| US-4 | As a user, I want to delete a note (with confirmation). | Must |
| US-5 | As a user, I want tags written as `#word` in the text to be found automatically. | Must |
| US-6 | As a user, I want to see all notes with a tag (`tag python`). | Must |
| US-7 | As a user, I want to search notes by a word in the title or text, not case-sensitive. | Must |
| US-8 | As a user, I want notes to be saved between runs. | Must |
| US-9 | As a user, I want to edit a note (replace the text or append lines). | Should |
| US-10 | As a user, I want to see all tags with the number of notes for each. | Should |
| US-11 | As a user, I want to export a note or all notes with a tag to `.md` files. | Should |
| US-12 | As a user, I want search results ranked: matches in the title first, then by the number of matches. | Could |
| US-13 | As a user, I want to see statistics: number of notes, words, the most used tag. | Could |
| US-14 | As a user, I want to share notes with friends online. | Won't |

### Acceptance criteria (examples)

**US-5**
- The text "Read about #Recursion and #python-basics today" has the tags `recursion` and `python-basics` (lower-case, without `#`).
- `#` inside a word (`C#`, `a#b`) is **not** a tag. A tag starts after a space or at the start of a line.

**US-7**
- Searching `recur` finds a note with "Recursion" in the text.
- Results show the ID, the title and a **snippet**: ~30 characters around the first match, with the match in `[brackets]`.

**US-11**
- `export 4` creates `export/4-lecture-recursion.md` (a "slug" from the title: lower-case, spaces → `-`, only letters, digits, `-`).
- An existing file is not overwritten silently — a number is added: `...-2.md`.

### MVP scope

**MVP = US-1 … US-8.** Release goal: + US-9, US-10, US-11.

---

## Stage 2. Project Manager — Project Plan

| Sprint | Goal | Stories | Milestone |
|--------|------|---------|-----------|
| 0 | Setup | — | repo, board, storage format decided by TL |
| 1 | "Write and read" | US-1, US-2, US-3, US-8 | notes survive a restart |
| 2 | "Find anything" | US-4, US-5, US-6, US-7 | **MVP** → Staging |
| 3 | "Organise and export" | US-9, US-10, US-11 (+ US-12, US-13) | **Release v1.0.0** |

### Risks

| Risk | Plan |
|------|------|
| Text with quotes, new lines, Unicode breaks the file | TL: JSON with `ensure_ascii=False`, UTF-8 everywhere |
| Search gets slow with many notes | TL: an index "tag → set of IDs"; measure with 10 000 generated notes |
| Tag rules are understood differently | PO writes examples; QA turns them into tests first |

---

## Stage 3. Tech Lead — Technical Overview

### Structure

```
notes/
├── main.py         # REPL and commands
├── note.py         # class Note
├── notebook.py     # class Notebook: CRUD, tags index, search
├── text_tools.py   # extract_tags, slugify, snippet
├── storage.py      # notes.json, UTF-8, atomic save
├── exporter.py     # .md export
└── tests/
```

### Data model

```python
class Note:
    id: int
    title: str
    body: str
    created: str          # ISO date-time
    updated: str
    @property
    def tags(self) -> set[str]: return extract_tags(self.body)

class Notebook:
    notes: dict[int, Note]
    tag_index: dict[str, set[int]]   # "python" → {1, 4, 7}
    add / get / update / delete      # keep tag_index in sync!
    by_tag(tag) -> list[Note]
    search(query) -> list[tuple[Note, str]]   # (note, snippet)
```

### Key algorithms

**Tag extraction** (Chapter 7): split the text into words by whitespace;
a word that starts with `#` and has at least one letter after it is a
tag; remove punctuation at the end (`#python,` → `python`); lower-case.

**Search:** `query.lower() in (title + "\n" + body).lower()` — linear in
the total text size. For the Could story US-12, count matches with
`str.count` and sort with a key `(-in_title, -count, -note.id)`.

**Tag index:** `by_tag` is `O(k)` where `k` is the number of notes with
that tag, instead of `O(n)` over all notes. Rebuild the index on load;
update it on add/edit/delete. This is the Chapter 15 discussion of the
project — QA measures both versions on 10 000 notes.

### Commands

```
new                  — asks for the title, then the text, ends with "."
list                 show <id>        delete <id>
edit <id>            append <id>
tag <name>           tags
find <words>
export <id> | export tag <name>
stats                help             quit
```

### Language notes

- **C++:** `std::map<int, Note>`, `std::unordered_map<std::string, std::set<int>>`;
  store notes as files `notes/<id>.txt` with a header line for the title.
- **JavaScript:** `Map` and `Set`; `String.prototype.includes` for search.

---

## Stage 4. Task List

| ID | Task | Story | SP | Depends on | Sprint |
|----|------|-------|----|------------|--------|
| NOTE-1 | Repo, structure, storage format document | — | 1 | — | 0 |
| NOTE-2 | `Note` class, `Notebook.add/get` | US-1, US-3 | 2 | NOTE-1 | 1 |
| NOTE-3 | `new` command: multi-line input until `.` | US-1 | 2 | NOTE-2 | 1 |
| NOTE-4 | `list` (newest first) and `show` | US-2, US-3 | 2 | NOTE-2 | 1 |
| NOTE-5 | `storage.py`: JSON, UTF-8, atomic save, broken file | US-8 | 3 | NOTE-2 | 1 |
| NOTE-6 | `extract_tags()` with all PO examples as tests | US-5 | 3 | NOTE-1 | 2 |
| NOTE-7 | Tag index + `tag <name>` command | US-6 | 3 | NOTE-6 | 2 |
| NOTE-8 | `find` with snippets | US-7 | 3 | NOTE-2 | 2 |
| NOTE-9 | `delete` with confirmation; index stays in sync | US-4 | 2 | NOTE-7 | 2 |
| NOTE-10 | Unit tests: notebook, search, storage | US-1…8 | 3 | NOTE-5, NOTE-8 | 2 |
| NOTE-11 | `edit` / `append`; `updated` time; index sync | US-9 | 3 | NOTE-9 | 3 |
| NOTE-12 | `tags` command with counts, sorted | US-10 | 1 | NOTE-7 | 3 |
| NOTE-13 | `slugify()` + export one note / a tag; no overwrite | US-11 | 3 | NOTE-7 | 3 |
| NOTE-14 | Ranked search | US-12 | 2 | NOTE-8 | 3 |
| NOTE-15 | Performance check: 10 000 generated notes, with and without the index | — | 2 | NOTE-7 | 3 |
| NOTE-16 | README, release notes | — | 1 | NOTE-10 | 3 |

**MVP total:** 24 SP (NOTE-1 … NOTE-10).

---

## Stage 5. Testing

### Unit tests

```python
from text_tools import extract_tags, slugify, snippet
from notebook import Notebook

def test_tags():
    assert extract_tags("Read about #Recursion and #python today") == {"recursion", "python"}
    assert extract_tags("I like C# and a#b") == set()
    assert extract_tags("#exam, #exam!") == {"exam"}
    assert extract_tags("") == set()

def test_slugify():
    assert slugify("Lecture: Recursion (part 2)") == "lecture-recursion-part-2"
    assert slugify("   ") == "note"          # never an empty file name

def test_index_in_sync():
    nb = Notebook()
    n = nb.add("A", "text #x")
    assert [m.id for m in nb.by_tag("x")] == [n.id]
    nb.update(n.id, body="text #y")
    assert nb.by_tag("x") == []
    nb.delete(n.id)
    assert nb.by_tag("y") == []

def test_search_case_insensitive():
    nb = Notebook()
    nb.add("Algo", "Recursion is fun")
    results = nb.search("recur")
    assert len(results) == 1 and "[Recur" in results[0][1]
```

### Manual test cases

| ID | Steps | Expected |
|----|-------|----------|
| TC-1 | `new`, title "Test", 3 lines, `.` | note created, ID shown |
| TC-2 | `new`, empty title | "The title cannot be empty" |
| TC-3 | `show 1` | full text with line breaks preserved |
| TC-4 | Note with `#Python` and `#python` | one tag `python` |
| TC-5 | `tag python` | only notes with that tag |
| TC-6 | `find RECURSION` | finds "recursion" with a snippet |
| TC-7 | `find zzz` | "Nothing found" |
| TC-8 | `delete 1`, answer `n` | the note is still there |
| TC-9 | Text with quotes `"`, emoji, Cyrillic `Привіт` | saved and shown correctly after a restart |
| TC-10 | `export 1` twice | two files: `...md` and `...-2.md` |
| TC-11 | `edit 1` removes a tag | `tags` no longer shows it (if no other note uses it) |

### Performance report (NOTE-15)

```
10 000 notes, 50 different tags
tag python   with index:  0.0002 s
tag python   without:     0.0150 s
find recur   (linear):    0.0400 s
```

The numbers above are only a format example — QA puts the team's real
measurements into the test report.

---

## Stage 6. Staging

1. `v1.0.0-rc1`; code freeze; fresh clone.
2. QA: unit tests, TC-1 … TC-11, the performance run.
3. **UAT:** the PO moves **at least 10 real notes** (lecture notes from
   this course!) into the app, tags them, searches, exports — and checks
   the Must stories.
4. **Data check:** QA opens `notes.json` in an editor and checks that
   Unicode is readable (not `аб...`).

### Staging checklist

- [ ] fresh clone works on Windows and macOS/Linux (UTF-8 issues!)
- [ ] all unit tests pass
- [ ] tag rules match every PO example
- [ ] no open critical / major bugs

---

## Stage 7. Production

1. PR `staging → main`, tag `v1.0.0`, GitHub Release.
2. Another team uses the app to keep notes for their own project
   meetings and reports bugs as GitHub issues.
3. Hotfix if needed → `v1.0.1`. Retro.

### Release notes (example)

```
## Notes v1.0.0

New:
  - multi-line notes, #tags found automatically
  - search by word with snippets, tag filter, tag list
  - export to Markdown files
Known issues:
  - BUG-5 (minor): very long lines are not wrapped in `show`
How to run:
  python3 main.py
```

---

## Event Cards for This Project

| Card | Text from the Client |
|------|----------------------|
| **Pinned notes** | "I want some notes always on top." — small story; the sort key changes. |
| **Encoding bug** | "On Windows my Ukrainian notes turned into `ÐŸÑ€Ð¸`!" — critical bug → hotfix: `encoding="utf-8"` everywhere. |
| **Import** | "I have 200 old `.txt` files. Can you import them?" — PO: big story; TL: the first line is the title. |
| **Search with two words** | "`find python exam` finds nothing, but I have such a note!" — unclear story: AND or exact phrase? PO decides. |

---

## Grading

The team grade follows [the common table](00-dev-flow.md#8-grading).
**Minimum for grade 3:** MVP works from a fresh clone; tags and search
follow the PO's examples; Unicode notes survive a restart.
