# HDBSCAN Single-Pass Variant

## Overview

This repository contains a custom implementation of the HDBSCAN clustering algorithm that performs hierarchy construction, persistence calculation, and cluster pruning in a **single pass** over the Minimum Spanning Tree (MST). This contrasts with the original HDBSCAN, which requires 3 traversals.

Key features:
* Efficient, single-pass design
* Supports high-dimensional data
* Matches clustering output of canonical HDBSCAN
* Identifies persistent clusters and noise

## Example
For 2d data X_train  (-1 cluster represents noise)

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




## Explanations

### Original HDBSCAN Workflow (Three-Pass Algorithm)

HDBSCAN typically proceeds through three conceptual passes over the data and the minimum spanning tree:

1. **Upward Pass – Hierarchy Construction**

   * Start with a mutual reachability distance matrix derived from k-nearest neighbor distances.
   * Build a minimum spanning tree (MST) from this matrix.
   * Traverse the MST edges in order of increasing distance to build a cluster hierarchy (dendrogram), recording `lambda_birth` and `lambda_death` for each cluster.

2. **Downward Pass – Persistence Calculation**

   * Once the full hierarchy is built, perform a top-down traversal to calculate the persistence (stability) of each cluster.
   * Persistence is based on how long a cluster survives in the hierarchy before being merged.

3. **Final Upward Pass – Pruning and Cluster Selection**

   * In a final bottom-up pass, compare each cluster’s persistence to the sum of its children’s.
   * Retain the cluster if it has higher persistence; otherwise, retain its children.
   * Assign points to the most stable clusters and label remaining points as noise.

### Single-Pass HDBSCAN Variant

This implementation integrates all three phases into a **single traversal** of the MST:

* As each pair of clusters is merged, their `lambda_birth` is set, and their persistence is computed on the spot.
* Immediately after merging, the algorithm decides whether to retain the newly formed cluster or its children by comparing persistences.
* Collapsed subclusters are discarded immediately, minimizing memory usage.
* This approach avoids explicit post-traversals, making it more efficient while preserving the core behavior of standard HDBSCAN.

By evaluating and collapsing clusters incrementally, this variant replicates the effect of HDBSCAN’s pruning in real-time during the MST edge iteration.

