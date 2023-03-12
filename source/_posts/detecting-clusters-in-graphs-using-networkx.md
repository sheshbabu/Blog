---
title: Detecting Clusters in Graphs using NetworkX
description: Clusters of all connected nodes inside a graph is commonly known as “connected components”
date: 2023-03-12 21:43:07
keywords: Python, Entity Resolution, NetworkX, Connected Components
tags:
  - Python
  - Entity Resolution
  - Graph
  - NetworkX
---

Clusters of all connected nodes inside a graph is commonly known as “connected components”

![](/images/2023-detecting-clusters-in-graphs-using-networkx/graph-clusters-connected-components.png)

Let’s recreate the above graph in NetworkX:

```python
import networkx as nx

G = nx.Graph()

G.add_nodes_from(["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N"])

G.add_edges_from([("B", "A"), ("E", "F"), ("E", "D"), ("G", "H"), ("I", "H"), ("J", "I"), ("J", "K"), ("M", "L")])
```

We then call the [connected_components](https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.components.connected_components.html) function of NetworkX.

```python
list(nx.connected_components(G))
```

Which will give the list of all clusters or connected components of this graph:

```python
[{'A', 'B'},
 {'C'},
 {'D', 'E', 'F'},
 {'G', 'H', 'I', 'J', 'K'},
 {'L', 'M'},
 {'N'}]
```
