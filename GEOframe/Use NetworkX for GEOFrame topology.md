---
owner: "Daniele Andreis"
created: 2025-11-14 08:43
updated: 2025-11-14
framework: GEOframe
tags: [hydrology, geoframe, network, python]
---

A basin network can be seen as a graph, specifically as a **directed acyclic tree**. In **GEOframe**, the topology of the basins, which describes the direction of the flow and the interconnections of the basins, is usually saved in a text file  with two columns: the first column represents the *origin basin*, and the second column represents the *target basin*. The **0** is used to mark the *outlet* of the network. For example:

> 
> 
> 
> 8 7
> 7 4
> 6 5
> 5 4
> 4 3
> 3 2
> 2 1
> 1 0
> 

The 0 mark the outlet.

---

In my work, I've encountered the need to manipulate basin networks so verifying the topological correctness of these new graphs is crucial. To accomplish this task, I chose the [[https://networkx.org/]] Python library. This library facilitates a variety of tasks—such as cutting the network, iterating through the network, and comparing two or more networks within the same watershed—thanks to its class structure, specifically the `Graph` class. As a result, I've developed some functions for basic operations on the network and shared them in my fork of [[https://github.com/bubbobne/GEOframePy/tree/network_utils]] on GitHub.

I've added two packages to this library. The first one, named `network`, includes functions to manipulate the network and check its integrity. In the `subbasin_check`, I've placed a function to check if the centroid of a basin is within the corresponding polygon.

The output of my functions is typically a `Graph` object, which enables the application of all functions available in NetworkX.  Note that a node is defined by its ID as a string.

---

## Create a network from a GEOFrame topology file

---

A NetworkX graph can be created from a GEOFrame topology file using two options: `get_raw_network` and `get_network`. In common scenarios, I recommend the use of `get_network` because this function includes several checks on the topology, such as verifying if the graph is acyclic or directed. However, in certain cases where these checks are unnecessary or fail (for example, with dams that pump water upstream), `get_raw_network` should be used instead.

Here, I've listed the implemented functions and provided some examples:

### Implemented Functions

- `check_network(net)`: Checks the topology of the graph.
- `get_upstream_network(net, node)`: Returns the upstream network from the provided node.
- `get_subgraph(net, gauge, gauge_id, is_calibration)`: Builds a network for a provided basin. I use it to extract the sub-network in the case of sequential calibration of the GEOFrame rainfall-runoff model and build sim files. If is_calibration is set to true then the nodes upstream to other gauge are deleted.
- `write_network(net, topology_path)`: Writes the network as a GEOFrame topology file.
- `sort_node(net, nodes)`: Retrieves the node with a topological order. Provide a list of nodes and the function return the it in the topological order.
- `get_order_node(topo_path, dict_path)`: Retrieves the topological order of the stream gauge.
- `simplify_network(topology_path, output_path):` Create a network to use for parallelization for kriging and other module.
- `get_downstream_network(net, basin_id)`: Return the network equal to net - get_upstream_network

To get the network:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Wed Apr 10 20:12:25 2024

@author: Daniele Andreis
"""
import geoframepy.topology.network as net

G = net.get_network('./topology.txt')
# you can iterate over the nodes
for node in G:
    print(node)
    
# get the node in the topological order
ordered = net.sort_node(G, ['2','4', '6'])
print(ordered)

# Extract the upstream, note that the outlet is the node "4"
G2 = net.get_up_streem_network(G, '4')
net.write_network(G2, './topology_from_4.txt')

# Write the network and add the outlet '0'
G3 = net.get_network('./topology_from_4.txt')

```

Using the function draw_networkx of NetworkX it;’s possible to plot the Tree.

![[GEOframe/Use NetworkX for GEOFrame topology/graph.png]]

G1 network

![[graph2.png]]

G2 network

![[graph3.png]]

G3 network

If you change the line where 2 go in 1 with 2 go in 3 you get this error:

```python
Attention the graph isn't acycling graph
Cycles in the graph:
('3', '2', 'forward')
('2', '3', 'forward')
```

---

To split the network for calibration purposes, we assume there are two stream gauges at node 2 and one at node 4:

```python
gauge = {'a':'2','b':'4'}
G4 = net.get_subgraph(G, gauge, "a", True)
G5 = net.get_subgraph(G, gauge, "b", True)
```

![[graph3 1.png]]

G4 network

![[graph5.png]]

G5 network

### Not Yet Implemented

- `plot_network(net, path=None)`:

## Check the Basin Centroid

Sometimes, it happens that the basin centroid is outside the polygons, especially when the polygon is non-convex. In such cases, it can result in model outputs being assigned a no-value (for instance, radiation values if the calculations are based on a DEM cropped to the polygon). To address this issue, I've written a function that checks for this situation. It then creates a new point based on the "centroid of the network" to ensure it lies inside the polygon.

The function is def check_centroid(path, crs_code, do_new_point): where path is the path where all subbasins pat are, crs_code of the shape file, do_new_point true if you want to create a new centroid, otherwise only write the problematic one.

### `check_centroid` Function

<aside>
⚠️ **If you create a new shapefile, be aware that the centroid values in the "subbasins.csv" file might refer to the old centroids. You will need to manually update these values in the new file.**

</aside>

The `check_centroid` function is designed to evaluate if the centroid of subbasins falls outside their respective polygons, particularly in cases of non-convex shapes. It can optionally create a new centroid within the polygon to ensure valid positioning for model calculations.

```python
def check_centroid(path, crs_code, do_new_point):
    """
    Checks the centroid of subbasins and optionally creates a new centroid.

    Parameters:
    - path (str): The directory path containing all subbasin folder.
    - crs_code (str/int): The coordinate reference system (CRS) code of the shapefiles.
    - do_new_point (bool): If True, creates a new centroid inside the polygon for any problematic subbasin. If False, only identifies and reports the problematic centroids.

    Returns:
    - A list of problematic subbasins with their original centroids if `do_new_point` is False.
    - A list of problematic subbasins with their new, corrected centroids if `do_new_point` is True.
    """
```

### Parameters:

- **path**: This is the directory path where all the subbasin folder are located. It should be provided as a string.
- **crs_code**: The CRS code of the shapefiles, which could be either a string (e.g., 'EPSG:4326') or an integer (e.g., 4326), depending on the format used by the shapefiles.
- **do_new_point**: A boolean flag indicating whether a new centroid should be created within the polygon for subbasins with centroids outside their polygon. Set this to `True` if you want the function to create a new centroid; otherwise, set it to `False` to only identify and report problematic subbasins.

### Returns:

This function returns a list of tuples, each representing a problematic subbasin. Each tuple contains the subbasin's identifier and the coordinates of the original or new centroid, depending on the value of `do_new_point`.

## Create NaN files for simulations:

To create a timeseries file with all missing values for each subbasin  and place it in the right folder, we developed a function that processes a network and generates these files automatically. This function reads a network, iterates over each subbasin node, and creates a timeseries 
DataFrame filled with `-9999.0` to indicate missing values.  Each timeseries file is then saved in a designated folder corresponding  to its subbasin ID, ensuring organized storage and easy access for  further analysis.

```python

from geoframepy.timeseries.files_utils import process_network_to_timeseries
process_network_to_timeseries(net, root_path, "2024-01-01 00:00", "2024-01-01 10:00", "H")
```

---

<aside>
💡

## Try it:

```bash
git clone git://github.com/bubbobne/GEOframePy.git
cd GEOframePy
git checkout network_utils
pip install .
```

</aside>

---

<aside>
🔥 To plot the network you have to install some library ( **PyGraphviz**, **pydot**), then use this snippets:

```python

import matplotlib.pyplot as plt
from networkx.drawing.nx_pydot import graphviz_layout
import networkx as nx

plt.figure(figsize=(10, 20))

for node in G5:
    if node == '0':
        color_map.append('red')
    else: 
        color_map.append('green')
            
pos = graphviz_layout(G5, prog="dot")

nx.draw_networkx(G5,pos, node_size=5200,font_size=30, node_color=color_map,alpha=0.5,  arrows=True,   arrowstyle="->", arrowsize=10)
plt.savefig("graph5.png")
plt.show()
```

</aside>

## References

- The NetworkX documentation: [[https://networkx.org/documentation/stable/]]