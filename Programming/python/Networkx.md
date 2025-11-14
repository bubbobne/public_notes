---
owner: Daniele Andreis
created: November 08, 2025 – 8:43 AM
updated: 2025-11-08
tags: [python, networkx, graph]
---

***Networkx*** is a python library to manage graph. The river network can be modelled as a directed tree graph.

## Create the network

```python
edges = []
    with open(topology_file, 'r') as data:
        for line in data:
            p = line.split()
            edges.append([p[0], p[1]])
    G = nx.DiGraph()
    G.add_edges_from(edges)
    if not nx.is_directed(G):
        print("Attention the graph isn't directed")
```

## Add field

```python

```