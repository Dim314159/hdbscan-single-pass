# HDBSCAN Single-Pass Variant

## Overview

This repository contains a custom implementation of the HDBSCAN clustering algorithm that performs hierarchy construction, persistence calculation, and cluster pruning in a **single pass** over the Minimum Spanning Tree (MST). This contrasts with the original HDBSCAN, which requires 3 traversals.

Key features:
* Efficient, single-pass design
* Supports high-dimensional data
* Matches clustering output of canonical HDBSCAN
* Identifies persistent clusters and noise

## Examples

X_train = varied_blobs_data
n_points = X_train.shape[0]

mst_agent = MST(number_of_nearest_neighbors=4)
mst_edges = mst_agent.get_mst_edges(X_train)


ds_agent = DisjointSet(
    n_points=n_points,
    min_cluster_size=5
)

ds_agent.build_hierarchy(mst_edges)

clusters_dict = ds_agent.get_clusters_dict()
n_plots = len(clusters_dict) + 1
fig, axis = plt.subplots(n_plots, 1, sharex=True, sharey=True, figsize=(6, n_plots * 3))
x = X_train[:, 0]
y = X_train[:, 1]
axis[0].scatter(x, y)
axis[0].set_title('Original Data')
for i, (cid, nodes) in enumerate(clusters_dict.items()):
    x = X_train[nodes, 0]
    y = X_train[nodes, 1]
    axis[i+1].scatter(x, y)
    axis[i+1].set_title(f'Cluster {cid}')
plt.show()
