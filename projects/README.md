# Projects

Bigger things I build with what I learn in `problems/` and `tp/` — something
that runs end to end, not an exercise with an expected answer.

| # | Project | Guide | Status |
|---|---|---|---|
| 1 | Org Chart Manager (Flask + SQLite web app) | [PROJECT_1_GUIDE.md](PROJECT_1_GUIDE.md) | not started |
| 2 | [Plant Disease Cascade](../../ClassificationDesPlantes/plantdisease/) (PyTorch, CNN, 25 leaf classes) | [PROJECT_2_GUIDE.md](../../ClassificationDesPlantes/PROJECT_2_GUIDE.md) | not started |

**Project 1 starts after** TP1 (`../tp/TP_OrgChart.ipynb`) and #102 / #103.
It reuses the same `../data/company.json` as seed data.

**Project 2 starts whenever** - it depends on nothing else here. It is a rebuild
of the June 2025 Keras project, in PyTorch, with the 2025 results kept in
`../../ClassificationDesPlantes/plantdisease/baseline_2025/` as the number to beat (**89.42%** end to end). Its
images are in `../../ClassificationDesPlantes/plantes/` - 9 MB sample committed, the 542 MB full split
gitignored.

Each project gets its own folder here (`orgchart/`, `../../ClassificationDesPlantes/plantdisease/`, ...), with
its data in `../data/`.

## Ideas for later

- **File-tree explorer** — a folder *is* a tree. Walk a real directory, report
  its depth, total size, biggest sub-folder. Same algorithms, real data.
- **JSON pretty-printer** — recurse through any nested JSON and print it
  indented. Recursion on a structure you did not design.
