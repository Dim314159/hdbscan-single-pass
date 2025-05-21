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
HDBSCAN begins by building a Minimum Spanning Tree (MST) using a mutual reachability distance matrix derived from k-nearest neighbor distances.

Then it proceeds through three conceptual passes:

1. **Upward Pass – Hierarchy Construction**

   * Traverse the MST edges in order of increasing distance.
   * Build a cluster hierarchy (dendrogram), assigning `lambda_birth` and `lambda_death` to each cluster.

2. **Downward Pass – Persistence Calculation**

   * Once the full hierarchy is built, perform a top-down traversal to calculate the persistence (stability) of each cluster.
   * Persistence is based on how long a cluster survives in the hierarchy before being merged.

3. **Final Upward Pass – Pruning and Cluster Selection**

   * In a final bottom-up pass, compare each cluster’s persistence to the sum of its children’s.
   * Retain the cluster if it has higher persistence; otherwise, retain its children.
   * Assign points to the most stable clusters and label remaining points as noise.

### Single-Pass HDBSCAN Variant
This implementation retains the initial MST construction step, but integrates all three conceptual passes into a single traversal over the MST edges.

The key ideas are:
   * Each cluster starts with only a `lambda_death` (think of time running backward).
   * When two clusters are merged into a **new** cluster:
       * Each of the two input clusters receives its `lambda_birth`, and its persistence is computed immediately.
       * Each of the two input clusters go through **collapsing**. Let’s call one of them **Cluster A**.
   * **Collapse logic**:
       * For **Cluster A**, we compare its persistence to the total persistence of its children (if any).
       * This decision is valid now because no future merges can change the internal relationship between **Cluster A** and its children.
       * If **Cluster A** wins, it absorbs both its children (and any descendants recursively), and all descendats are pruned from the hierarchy.
       * If not, **Cluster A** is marked as noise (remains in the tree), and its children are retained.
   * Later, the **new** cluster (parent of **Cluster A**) may itself be merged, triggering the same evaluation and **collapse** process resulting in absorbing **Cluster A** (and its children if any) or leaving it (and its children if any) intact.

This recursive, local pruning makes the algorithm memory-efficient and conceptually clean—collapsing happens immediately when a cluster’s future is sealed.

By collapsing clusters as they become finalized, this variant replicates the pruning behavior of standard HDBSCAN in real time—without needing post-processing passes.
