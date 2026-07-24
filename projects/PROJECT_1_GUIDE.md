# Project 1 — Org Chart Manager

**Start it after:** TP1 (`tp/TP_OrgChart.ipynb`) and #102 / #103.

A small web app that manages a company's org chart: hire people, give raises,
move someone to a new manager, fire someone, and look at the whole tree — its
depth, its payroll, the cost of any team.

You already wrote every algorithm it needs. This project is about everything
*around* the algorithm: storing it, exposing it, and structuring the code so it
doesn't rot.

---

## Why this project

| You learned | Where it comes back |
|---|---|
| tree + recursion | depth, payroll, rendering the chart |
| BFS + batches (#104/#102) | "show the company level by level" |
| dict index | `id -> employee`, built once per request |
| `json` | import/export, the seed data |
| your `Queue` | still works — or swap in `deque` |
| **new:** POO | `Employee`, `Repository`, `Service` |
| **new:** CRUD | the four things a user can actually do |
| **new:** SQL / database | the tree, stored so it survives a restart |
| **new:** design patterns | Repository, Service layer, Factory |

---

## Stack — deliberately tiny

- **Python 3** — you know it
- **Flask** — `pip install flask`. One file to start. Minimal magic.
- **SQLite** via the built-in `sqlite3` module — a whole database in one file,
  nothing to install, nothing to run
- **Jinja2** templates — comes with Flask
- **No ORM at first.** Write the SQL yourself. Later, if you want, redo the data
  layer with SQLAlchemy — you'll only appreciate what an ORM does after you've
  done it by hand.

No React, no Docker, no login system. Not yet. Ship the small thing first.

---

## The big idea: a tree inside a table

A table is flat. A tree is not. So how do you store one in the other?

**You store the parent pointers.** Exactly the `boss = {name: manager}` dict from
TP exercise 6 — except now it lives on disk.

```sql
-- schema.sql
CREATE TABLE employee (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    name       TEXT    NOT NULL,
    title      TEXT    NOT NULL,
    salary     INTEGER NOT NULL CHECK (salary >= 0),
    manager_id INTEGER REFERENCES employee(id)   -- NULL only for the CEO
);
```

That `manager_id` pointing back at `employee(id)` is called a **self-referencing
foreign key**, and the layout is called an **adjacency list**. One row per
person, one pointer each. That is the entire tree.

Then in memory you rebuild the children lists — and that is TP exercise 7 again:

```python
# skeleton, not the answer
def build_tree(rows):
    # 1. one pass: make an Employee per row, index them by id      -> dict
    # 2. one pass: attach each employee to its manager's .reports  -> defaultdict helps
    # 3. return the one whose manager_id is None                   -> the root
```

Two passes, `O(n)`, no recursion needed to *build* it — recursion comes back when
you *walk* it. Think about why pass 1 must finish before pass 2 starts.

---

## Folder layout

```
projects/orgchart/
├─ app.py            Flask routes  (the web layer — thin!)
├─ db.py             connection handling, schema init
├─ models.py         Employee  (POO)
├─ repository.py     all the SQL lives here, and NOWHERE else
├─ services.py       the business rules
├─ schema.sql
├─ seed.py           loads ../../data/company.json into the DB
├─ templates/
│  ├─ base.html
│  ├─ index.html
│  ├─ employee.html
│  └─ _node.html     renders one node + recurses on its children
└─ static/style.css
```

The rule that makes this layout worth having: **`app.py` never writes SQL, and
`repository.py` never knows the web exists.** If you can't swap SQLite for a JSON
file by editing only `repository.py`, the layers have leaked.

---

## Build it in 5 steps

Each step runs and does something. Don't start the next one until the current
one works.

### Step 1 — Model, no web, no DB
`models.py`: an `Employee` class (`id, name, title, salary, reports`). Load
`data/company.json`, build the tree, and port your TP functions onto the class:
`count()`, `depth()`, `total_salary()`, `find(name)`.

**Done when:** a plain script prints `11 people, depth 4, payroll 733000`.

### Step 2 — Database
Write `schema.sql`. Write `seed.py` to walk the JSON tree and `INSERT` each
person with the right `manager_id`. (Question: in what order must you insert, and
why? What breaks if you insert a child before its manager?)

Write `repository.py` with the plain functions you'll need: `get_all()`,
`get(id)`, `add(...)`, `update(...)`, `delete(id)`.

**Done when:** `seed.py` fills `orgchart.db`, and `build_tree(repo.get_all())`
gives the same numbers as Step 1.

### Step 3 — CRUD in the terminal
A small menu loop. Hire, raise, reassign, fire, print the tree. Ugly is fine —
this is where you find out which operations are actually hard *before* you also
have HTML to worry about.

### Step 4 — Flask
Routes: `/` (the chart), `/employee/<id>`, and POST routes for the four
operations. Templates. Keep the route functions ~5 lines: read the form, call a
service, redirect.

### Step 5 — The tree views
- `/` renders the chart with `_node.html` including **itself** for each child —
  recursion, in a template. Your first non-Python recursion.
- `/levels` shows the company level by level — that's #102, rendered.
- Each employee page shows their team cost (TP exercise 5) and their chain of
  command up to the CEO (TP exercise 6).

---

## CRUD, and the decisions it forces you to make

This is the real content of the project. Each letter hides a design question.

**C — hire.** Every new person needs a manager. What if the manager id doesn't
exist? What if the table is empty (the very first employee is the CEO, with
`manager_id = NULL`)?

**R — read.** Fine. But: do you rebuild the whole tree on every page load, or
cache it? At 11 employees, who cares. At 100 000? Say what your answer depends
on.

**U — update.** Raising a salary is easy. **Changing someone's manager is the
interesting one.**

> What happens if you make Amine report to Nadia?

Amine is Nadia's great-grandparent. Now the parent pointers form a **cycle** —
and a tree with a cycle is not a tree. Your recursive `depth()` will follow it
forever and blow the stack. It's the exact failure mode of your `maxDepthV1`,
except caused by *data* instead of code.

So `services.py` has to refuse the move. How do you detect it? Walk from the
proposed new manager up the chain of command — if you meet the person you're
moving, the move creates a cycle. That's TP exercise 6 as a **validation rule**.
The database's `FOREIGN KEY` cannot catch this for you; only your code can.

**D — fire.** Someone with reports leaves. What happens to their team? There are
three defensible answers:

1. **refuse** the delete while they still have reports
2. **cascade** — delete the whole subtree (brutal, and rarely what a business wants)
3. **promote** — re-attach their reports to their own manager

Pick one deliberately and write down why. Do not let the database decide by
accident — look up what `ON DELETE CASCADE` vs `ON DELETE SET NULL` would do
here, and notice that `SET NULL` would silently create a **second root**. Is your
`build_tree` ready for a forest?

---

## Design patterns you will actually use

Learn these four. Ignore the other twenty for now.

**Repository** — the important one. All SQL in one module, behind function names
that talk about employees, not tables. The rest of the app asks for employees and
has no idea a database exists. Test: could you rewrite it to read JSON files
without touching any other file? If yes, you did it right.

**Service layer** — business rules that aren't SQL and aren't HTML. "No cycles."
"Salary can't be negative." "A manager can't be fired while they have reports."
This is where the app's *thinking* lives, and it's the part you'd keep if you
threw away the web UI tomorrow.

**Factory** — a small `Employee.from_row(row)` classmethod that turns a DB row
into an object. One place that knows the mapping, instead of that knowledge
smeared across the codebase.

**Singleton-ish connection** — one connection per request, reused within it, not
a fresh one per query. Flask has `g` and `teardown_appcontext` for exactly this.

> ⚠️ A pattern applied where there is no problem is worse than no pattern at all.
> Before you add one, be able to say the sentence: *"without this, X would be
> painful because Y."* If you can't finish the sentence, skip it.

---

## Collections you'll reach for

| Collection | Where |
|---|---|
| `dict` | `id -> Employee` index, built once |
| `collections.defaultdict(list)` | grouping rows by `manager_id` when rebuilding the tree |
| `collections.deque` | BFS for the `/levels` page |
| `dataclass` | `Employee`, if you want `__repr__` and `__eq__` for free |
| `collections.Counter` | headcount by title, for a small stats page |

---

## Milestones

- [ ] Step 1 — tree in memory, numbers match the TP
- [ ] Step 2 — SQLite schema + seeded from JSON
- [ ] Step 3 — CRUD from the terminal
- [ ] Step 4 — Flask, four operations through a browser
- [ ] Step 5 — recursive template, `/levels`, team cost, chain of command
- [ ] Cycle detection refuses an illegal reassignment
- [ ] A documented, deliberate answer to "what happens when a manager is fired"

## Stretch goals — only after all of the above

- `GET /api/tree` returning JSON, so the app can feed something else
- search by name, with the query going to SQL rather than a Python scan
- export the tree back out to `company.json` — round-trip it
- `pytest` tests for the service rules (cycle detection is a perfect first test)
- rewrite `repository.py` with SQLAlchemy and compare the two by hand
- swap the adjacency list for a **nested set** or **closure table** and find out
  what each one makes fast and what it makes slow

## Not yet

Authentication, Docker, React, deployment, migrations. Every one of those is a
project in itself. Finish this one first — a small thing that works beats a big
thing that doesn't.
