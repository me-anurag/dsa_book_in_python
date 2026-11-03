<div align="center">

<img src="media/banner.svg" alt="Arrays & Strings in Python: two pointers converging on a target sum" width="100%"/>

<img src="media/typing.svg" alt="Learn the LeetCode dialect of Python · 17 pattern chapters · 119 verified problems" width="90%"/>

<p>
  <img src="https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.10+"/>
  <img src="https://img.shields.io/badge/LeetCode-119%20problems-FFA116?style=for-the-badge&logo=leetcode&logoColor=black" alt="119 LeetCode problems"/>
  <img src="https://img.shields.io/badge/Chapters-32-0EA5E9?style=for-the-badge" alt="32 chapters"/>
  <img src="https://img.shields.io/badge/Format-Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white" alt="Markdown"/>
</p>

**A book that takes you from "knows basic Python" to mastery of arrays and strings<br/>
for LeetCode-style problems and coding interviews.**

</div>

<div align="center">
<img src="media/stats.svg" alt="32 chapters, 119 LeetCode problems, 17 pattern chapters, 5 appendices" width="100%"/>
</div>

## ✨ What's inside

| | |
|---|---|
| 🐍 **The LeetCode dialect of Python** | `class Solution`, `-> None` in-place contracts, type hints old and new, the semantics that silently break answers |
| 🧰 **The complete toolkit** | Every `str` and `list` method, the built-ins and stdlib that matter, and a tier map of what you must be able to **hand-write** |
| ⏱️ **Honest performance** | Complexity with a reason for every claim, and timings measured on real hardware instead of folklore |
| 🧩 **17 pattern chapters** | Each pattern goes brute force → optimal, with templates, invariants, dry runs and practice sets |
| 🎯 **Mastery** | Pattern recognition from the problem statement, mixed problem sets, mock interviews, and the 25 most common bugs |

Every chapter is plain Markdown and renders directly on GitHub. Start with the
**[Front Matter](src/00-front-matter.md)**: how the book is organized, the practice
method, and the re-solve schedule.

## 🗺️ Learning roadmap

```mermaid
flowchart LR
    F([Front Matter]) --> P1
    subgraph P1 [Part I · Python dialect]
        C1[Ch 1–4]
    end
    subgraph P2 [Part II · Toolkit]
        C5[Ch 5–9]
    end
    subgraph P3 [Part III · Performance]
        C10[Ch 10–12]
    end
    subgraph P4 [Part IV · Array patterns]
        C13[Ch 13–23]
    end
    subgraph P5 [Part V · String patterns]
        C24[Ch 24–29]
    end
    subgraph P6 [Part VI · Mastery]
        C30[Ch 30–32]
    end
    P1 --> P2 --> P3 --> P4 --> P5 --> P6
    P6 --> A([Appendices A–E])
```
