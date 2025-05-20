# HDBSCAN Single-Pass Variant

## Overview

This repository contains a custom implementation of the HDBSCAN clustering algorithm that performs hierarchy construction, persistence calculation, and cluster pruning in a **single pass** over the Minimum Spanning Tree (MST). This contrasts with the original HDBSCAN, which requires 3 traversals.

Key features:
* Efficient, single-pass design
* Supports high-dimensional data
* Matches clustering output of canonical HDBSCAN
* Identifies persistent clusters and noise

## Example
For 2d data X_train  

n_points = X_train.shape[0]  
mst_agent = MST(number_of_nearest_neighbors=4)  
mst_edges = mst_agent.get_mst_edges(X_train)  
ds_agent = DisjointSet(  
    n_points=n_points,  
    min_cluster_size=5  
)  
ds_agent.build_hierarchy(mst_edges)  
clusters_dict = ds_agent.get_clusters_dict()  

![image](https://github.com/user-attachments/assets/f401b8fc-02c1-41d0-8e03-ef76ecf27076)

