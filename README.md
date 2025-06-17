# HDBSCAN Single-Pass Variant

## Overview

This repository contains a custom implementation of the HDBSCAN clustering algorithm that performs hierarchy construction, persistence calculation, and cluster pruning in a **single pass** over the Minimum Spanning Tree (MST). This contrasts with the original HDBSCAN, which requires 3 traversals.

Key features:
* Efficient, single-pass design
* Supports high-dimensional data
* Matches clustering output of canonical HDBSCAN
* Identifies persistent clusters and noise


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
This implementation retains the initial MST construction step, but integrates all three conceptual passes into a **single traversal** over the MST edges.

The key ideas are:
   * Each cluster starts with only a `lambda_death` (think of time running backward).
   * When two clusters are merged into a **new** cluster:
       * Each of the two input clusters receives its `lambda_birth`.
       * Each of them go through **collapsing** (let’s call one of them **Cluster A**).
   * **Collapse logic**:
       * Compute persistence of **Cluster A**.
       * Compare its persistence to the total persistence of its children (if any).
       * This decision is valid now because no future merges can change the internal relationship between **Cluster A** and its children.
       * If **Cluster A** wins:
           * It absorbs both its children (and any descendants recursively).
           * All descendats are pruned from the hierarchy.
       * If children win:
           * **Cluster A** is marked as noise (remains in the tree).
           * The children are retained as active clusters.
   * Later, the **new** cluster (parent of **Cluster A**) will merge again, triggering the same **collapse** process on its children (including **Cluster A**) resulting in absorbing **Cluster A** (and its children if any) or leaving it (and its children if any) intact.

This recursive, local pruning makes the algorithm memory-efficient and conceptually clean: collapsing happens immediately when a cluster’s future is sealed.

By collapsing clusters as they become finalized, this variant replicates the pruning behavior of standard HDBSCAN in real time—without needing post-processing passes.


## How to use
### Parameters to tune:
min_cluster_size : int, default=5
 The minimum size of clusters; used to decide when a cluster is considered significant. Smaller clusters are treated as noise or merged into larger ones.

number_of_nearest_neighbors : int or None, default=None
 The number of neighbors used to define the core distance and compute mutual reachability. If None, it defaults to min_cluster_size. Must be ≤ min_cluster_size.

cpu_gpu_device : str, default='cpu'
 The device to run computations on. Options:
     'cpu' – all computations on CPU.
     'cuda' or 'cuda:0', 'cuda:1', etc. – use specific GPU. Requires PyTorch with CUDA.

backend : str or None, default=None
 The algorithm used to compute approximate nearest neighbors:
    None – Exact computation using PyTorch (torch.cdist).
     Best for small datasets (<10,000 points).
     Avoid if your machine runs out of memory during distance computation.
    'faiss' – Use Facebook’s FAISS for fast approximate nearest neighbors.
     Recommended if:
    You’re working with large datasets,
    You want GPU acceleration, or
    You need faster clustering with minimal accuracy tradeoff.
    FAISS supports both CPU and GPU,
    Requires separate installation (pip install faiss-cpu or faiss-gpu).
    'hnsw' – Use HNSWlib, a fast CPU-only alternative.
     Simpler to install than FAISS.
     Good option if you:
        Don’t have a GPU,
        Don’t want to install FAISS,
        Still want fast clustering on large datasets.

faiss_M : int, default=32
 Connectivity parameter for FAISS HNSW index. Higher values yield better accuracy at the cost of indexing time and memory. Allowed values: 16, 32, 48.

faiss_efConstruction : int, default=64
 Controls the effort during FAISS index construction. Higher values give better accuracy. Allowed values: 32, 64, 128.

faiss_efSearch : int, default=64
 Controls the search effort in FAISS queries. Higher values lead to more accurate k-NN results. Allowed values: 32, 64, 128, 256.

epsilon : float, default=1e-10
 A small numerical tolerance to prevent division-by-zero or instability in lambda-based computations.



🧠 Notes
M controls how many links are kept per node in the HNSW graph.
efConstruction controls quality of graph during index building.
efSearch controls how many nodes to explore at query time. Larger = higher recall.

### Example
Initiate class object:  
clusterer= HDBSCAN()
you can get labels by:  
clusterer.fit(X)  
labels = clusterer.labels_  

or immideately:
labels = clusterer.fit_predict(X)

(lebel '-1' represents noise)


![image](https://github.com/user-attachments/assets/f401b8fc-02c1-41d0-8e03-ef76ecf27076)




