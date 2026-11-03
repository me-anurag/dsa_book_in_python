# Part I · Python for LeetCode

> How a LeetCode solution is structured, how type hints work, and the Python semantics that silently break answers

## What this part contains

This part teaches the **dialect** of Python that LeetCode solutions are written in. It covers how the judge calls your `class Solution`, the `-> None` in-place contract, `List[int]` vs `list[int]`, and the language rules (aliasing, `//` with negatives, unbounded ints, truthiness, comprehensions) that cause plausible-looking wrong answers.

## When to read it

Read it first, even if you already know Python. Chapters 1 and 3 explain the bugs that fail the most submissions.

## Chapters

| Chapter | Title |
|---|---|
| 1 | [Anatomy of a LeetCode Solution](ch01-anatomy-of-a-leetcode-solution.md) |
| 2 | [Type Hints, Completely](ch02-type-hints-completely.md) |
| 3 | [Core Python Semantics That Decide DSA Correctness](ch03-core-python-semantics.md) |
| 4 | [Comprehensions & Generators](ch04-comprehensions-and-generators.md) |

---

[← Front Matter](../00-front-matter.md) · [Book home](../../README.md) · [Part II · The Python Toolkit →](../part2-python-toolkit/README.md)
