# LeetCode Practice

My problem-solving journey in Python. Started **2026-07-20**.

## Folder layout

```
Problems/
├─ README.md        this index
├─ tracker.json     the source of truth (status, difficulty, dates, complexity)
├─ problems/        one notebook per LeetCode problem
├─ tp/              practical exercises (TP) - bigger, multi-part, not from LeetCode
├─ learn_json/      learning track: the json module, from zero
├─ projects/        real projects built out of what I learned
└─ data/            input files the notebooks read (.json, .csv, .txt)
```

> The folder is `learn_json/`, **not** `json/`. A folder named `json` sitting next
> to your notebook would shadow Python's own `json` module and `import json`
> would import the folder instead. Never name a file or folder after a module
> you import.

Each notebook holds the problem statement in a markdown cell, then `class
Solution` in the code cell below it, then a test cell.

**Reading a data file from a notebook:** notebooks in `tp/` reach the data with
a relative path that goes up one level first, e.g.
`open("../data/company.json")`.

### `status` values in the tracker

- `todo` — file created, not started
- `in_progress` — attempted, not passing yet
- `solved` — accepted
- `review` — solved but I want to redo it later

## Progress

**10 solved / 12 attempted**

### Problems

| # | Problem | Difficulty | Status |
|---|---|---|---|
| 9 | [Palindrome Number](problems/PalindromeNumber.ipynb) | Easy | in_progress |
| 125 | [Valid Palindrome](problems/PalindromeString.ipynb) | Easy | ✅ solved |
| 1 | [Two Sum](problems/TwoSum.ipynb) | Easy | ✅ solved |
| 104 | [Maximum Depth of Binary Tree](problems/MaxDepthBinaryTree.ipynb) | Easy | ✅ solved |
| 102 | [Level Order Traversal](problems/LevelOrderTraversal.ipynb) | Medium | ✅ solved |
| 103 | [Zigzag Level Order Traversal](problems/ZigzagLevelOrder.ipynb) | Medium | ✅ solved |
| 242 | [Valid Anagram](problems/ValidAnagram.ipynb) | Easy | ✅ solved |
| 226 | [Invert Binary Tree](problems/InvertBinaryTree.ipynb) | Easy | todo |
| 206 | [Reverse Linked List](problems/ReverseLinkedList.ipynb) | Easy | ✅ solved |
| 114 | [Flatten Binary Tree to Linked List](problems/FlattenBinaryTree.ipynb) | Medium | ✅ solved |
| 105 | [Construct Binary Tree from Preorder and Inorder Traversal](problems/ConstructTreePreorderInorder.ipynb) | Medium | ✅ solved |
| 155 | [Min Stack](problems/MinStack.ipynb) | Medium | ✅ solved |

### TP

| TP | Topic | Data | Status |
|---|---|---|---|
| [Company Org Chart](tp/TP_OrgChart.ipynb) | tree + json + dict | [company.json](data/company.json) | in_progress |
| [Store Catalog](tp/TP_JsonCatalog.ipynb) | recursion over nested JSON | [catalog.json](data/catalog.json) | todo |

### Learning track - JSON

Do these **in order**, they build on each other. Paused the Org Chart TP until
this is done.

| # | Notebook | What it teaches | Data |
|---|---|---|---|
| 01 | [The 4 functions](learn_json/01_JsonBasics.ipynb) | `load` / `loads` / `dump` / `dumps`, the JSON↔Python type table | [student.json](data/student.json) |
| 02 | [List of objects](learn_json/02_JsonLists.ipynb) | the shape 90% of real JSON has: filter, `max(key=)`, sort, group, index | [products.json](data/products.json) |
| 03 | [Nested & missing keys](learn_json/03_JsonNested.ipynb) | walking chains, `KeyError`, `.get(k, default)`, chained `.get` | [orders.json](data/orders.json) |
| 04 | [Writing JSON](learn_json/04_JsonWrite.ipynb) | `indent`, `ensure_ascii=False`, what JSON can't store, string keys | [scores.json](data/scores.json) |
| 05 | [Errors & a messy file](learn_json/05_JsonErrors.ipynb) | `JSONDecodeError`, holes in real data, flattening to clean records | [library.json](data/library.json) |

### Projects

_Nothing yet._
