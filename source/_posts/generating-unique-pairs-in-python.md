---
title: Generating Unique Pairs in Python
date: 2023-03-12 21:33:59
keywords: Python, Entity Resolution, Candidate Pairs Generation, Itertools
tags:
  - Python
  - Entity Resolution
---

Let’s say we’ve a list of entities, this can be anything like people names, company names, etc from which we’d like to generate unique pairs

```python
entities = ["A", "B", "C", "D", "E"]
```

We can use the `distinct_combinations` function from [more_itertools](https://more-itertools.readthedocs.io/en/stable/index.html) package

```python
from more_itertools import distinct_combinations

candidate_pairs = distinct_combinations(entities, 2)
```

This will generate all the possible pairs with duplicates removed:

```python
[('A', 'B'),
 ('A', 'C'),
 ('A', 'D'),
 ('A', 'E'),
 ('B', 'C'),
 ('B', 'D'),
 ('B', 'E'),
 ('C', 'D'),
 ('C', 'E'),
 ('D', 'E')]
```
