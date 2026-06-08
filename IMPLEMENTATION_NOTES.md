# Technical Implementation Notes

## Architecture Overview

### Component Pipeline

```
01_download_osm.py
      ↓
    Directed graph (MultiDiGraph) from OSM
      ↓
02_build_csr.py
      ↓
    CSR format + UTM projection
      ↓
03_generate_queries.py
      ↓
    10 landmark queries + 1000 random queries
      ↓
    ├─→ cpu_seq_10.py      [CPU baseline]
    ├─→ cpu_seq_1000.py    [CPU baseline - scaled]
    ├─→ gpu_par_block_threads_10.cu     [GPU version]
    ├─→ gpu_par_block_threads_1000.cu   [GPU version - scaled]
    └─→ 04-06_visualize*.py [Results visualization]
```

---

## Data Structures

### 1. Graph Representation: Compressed Sparse Row (CSR)

**Why CSR?**

Consider the adjacency matrix approach:
- Hanoi network: ~45,000 nodes × 45,000 nodes
- Dense matrix: 45,000² ≈ 2 billion entries
- Memory: 2B entries × 4 bytes/float = **8 GB** (impractical)

Reality check - road networks are **sparse**:
- Average degree: ~2-4 neighbors per node (real-world streets)
- Actual edges: ~120,000
- CSR format: 45K + 120K = **165K entries** ≈ 0.66 MB

**CSR Format:**

Three arrays represent the graph:

```
row_ptr[i] = starting index in col_index/data for node i's neighbors
col_index[] = destination node IDs (length = # edges)
data[]      = edge weights/distances (length = # edges)
```

Example (3-node graph):
```
Edges: 0→1 (weight 5), 0→2 (weight 3), 2→1 (weight 2)

Dense format:
    0  1  2
0 [ ∞  5  3 ]
1 [ ∞  ∞  ∞ ]
2 [ ∞  2  ∞ ]

CSR format:
row_ptr    = [0, 2, 2, 3]      (size = V+1 = 4)
col_index  = [1, 2, 1]         (size = E = 3)
data       = [5, 3, 2]         (size = E = 3)

Access: neighbors(0) = col_index[0:2] = {1, 2}, weights = {5, 3}
        neighbors(1) = col_index[2:2] = {}       (no neighbors)
        neighbors(2) = col_index[2:3] = {1}      weights = {2}
```

**GPU Benefits:**
- **Coalesced Memory Access**: All threads in a warp can read contiguous memory
- **Cache Efficiency**: Sequential data layout maximizes cache hits
- **Memory Bandwidth**: Aligned access patterns saturate GPU bandwidth

---

## Algorithms

### A* Search Algorithm

**High-level pseudocode:**

```python
function A_Star(start, goal):
    open_set = PriorityQueue()    # min-heap by f-score
    open_set.push((h(start), start))
    
    g_score = {start: 0, others: ∞}
    f_score = {start: h(start), others: ∞}
    closed = {}
    
    while open_set is not empty:
        _, current = open_set.pop()
        
        if current == goal:
            return reconstruct_path(current)
        
        closed.add(current)
        
        for neighbor in neighbors(current):
            if neighbor in closed:
                continue
            
            tentative_g = g_score[current] + distance(current, neighbor)
            
            if tentative_g < g_score[neighbor]:
                # Found a better path
                parent[neighbor] = current
                g_score[neighbor] = tentative_g
                f_score[neighbor] = tentative_g + h(neighbor)
                open_set.push((f_score[neighbor], neighbor))
    
    return NOT_FOUND
```

**Why A* is optimal:**

1. **Admissibility**: h(n) ≤ actual_cost(n → goal)
   - Euclidean distance never overestimates road distance
   - Therefore: A* never skips the optimal path

2. **Consistency** (stronger): h(n) ≤ cost(n→m) + h(m)
   - Ensures A* processes each node at most once
   - With consistent heuristic: first path found is optimal

**Heuristic: Euclidean Distance**

```
h(n) = √((x_n - x_goal)² + (y_n - y_goal)²)
```

- **Admissible** ✓ (straight-line distance ≤ road distance)
- **Consistent** ✓ (satisfies triangle inequality in metric space)
- **Informedness**: Medium (better than Manhattan, worse than Landmark heuristics)
- **Computation**: O(1) per node

---

## GPU Parallelization Strategies

### Strategy 1: Block-per-Query (Current Implementation)

**Mapping:**

```
Query 0  → Block 0  → Thread 0 runs A* for Query 0
Query 1  → Block 1  → Thread 0 runs A* for Query 1
...
Query 999 → Block 999 → Thread 0 runs A* for Query 999
```

**Characteristics:**

- **Parallelism**: Task-level (across queries)
- **Per-block resources**: Each block has its own:
  - g_score[V]    (global memory)
  - parent[V]     (global memory)
  - open_list[V]  (global memory)
  - closed[V]     (global memory)
- **Synchronization**: None needed (blocks are independent)
- **GPU utilization**: 1,000 blocks active simultaneously (scalable)

**Memory per query (1 block):**

```
g_score:  45,000 × 4 bytes = 180 KB
parent:   45,000 × 4 bytes = 180 KB
open_list: 45,000 × 12 bytes = 540 KB  (struct with id, f_score)
closed:   45,000 × 1 byte = 45 KB
Total per query: ~945 KB

GPU memory budget: Typical GPU 6-24 GB
Concurrent queries: 6GB / 945KB ≈ 6,300 queries possible!
```

**Performance characteristics:**

- **Kernel launch overhead**: ~1-10 μs (amortized across V node expansions)
- **Memory bandwidth**: Each query reads full graph (120K edges worth of CSR data)
- **Cache utilization**: Good for CSR format (coalesced access)
- **Divergence**: Minimal (all threads execute same A* code path)

### Strategy 2: Multiple-Threads-per-Query (Future Optimization)

**Mapping:**

```
Query 0 → Block 0 → Threads 0-31 cooperatively solve Query 0
Query 1 → Block 1 → Threads 0-31 cooperatively solve Query 1
```

**Potential improvements:**

- Parallel neighbor exploration (threads process different neighbors)
- Parallel open list management (warp-wide reduction for min-heap operations)
- Reduced memory per query (shared memory collaboration)

**Tradeoffs:**

- Increased complexity (synchronization, shared memory management)
- Potential speedup: 2-8× depending on bottleneck
- Not implemented in current version (focus on task parallelism)

---

## Memory Bandwidth Analysis

### Why CSR is Critical for GPU Performance

**Example: Graph traversal for one A* node expansion**

```
Event: Expand node u (degree d = 4 neighbors)

Sequential (CPU):
  for v in neighbors(u):        # Read 4 node IDs from col_index
    edge_weight = ...           # Read 1 weight
    Total reads: 4 × (4 + 4) = 32 bytes

GPU warp (32 threads, 8 queries):
  Thread 0 reads: neighbor 0, weight 0
  Thread 1 reads: neighbor 1, weight 1
  ...
  Thread 7 reads: neighbor 7, weight 7
  Total coalesced read: 1 cache line (64 bytes) per array
  Effective bandwidth: 32 threads × 32 bytes / latency
```

**Bandwidth comparison:**

```
Dense adjacency matrix:
  - Random access to 45K × 45K matrix
  - Cache miss rate: ~95% (too large for GPU cache)
  - Wasted bandwidth: Reading full rows when only 2-3 entries needed
  - Effective throughput: 10-20% of peak GPU bandwidth

CSR format:
  - Sequential access to col_index and data arrays
  - Cache hit rate: ~80% (hot path in GPU cache)
  - Wasted bandwidth: 0 (read exactly what needed)
  - Effective throughput: 70-90% of peak GPU bandwidth
```

---

## Benchmark Metrics

### Measured Quantities

1. **Execution Time** (milliseconds)
   - Total time for all queries
   - Includes kernel launch overhead
   - Includes PCIe data transfer (if applicable)

2. **Nodes Expanded**
   - How many nodes did A* visit?
   - Indicator of algorithm efficiency (not implementation)
   - Should be same for CPU and GPU (both run same algorithm)

3. **Speedup**
   - GPU time / CPU time
   - 1.0× = no benefit
   - 3.0× = GPU is 3× faster
   - Expected: 3-10× for large batch sizes

4. **Throughput**
   - Queries per second
   - Scaled with batch size
   - Shows when GPU becomes efficient

### Interpretation Guidelines

**Small batches (1-10 queries):**
- CPU may be faster! (GPU launch overhead dominates)
- GPU speedup < 1.0× is normal and expected
- GPU is not suitable for single queries

**Medium batches (50-100 queries):**
- GPU starts to win
- Speedup: 1.5-3.0×
- Break-even point varies by hardware

**Large batches (500-1000 queries):**
- GPU significantly faster
- Speedup: 5-15×
- Amortized overhead becomes negligible

---

## Coordinate Systems

### OSM Coordinates: Lat/Lon (EPSG:4326)

```
OSM native: degrees
Example: (21.0285°N, 105.8523°E) - Hồ Hoàn Kiếm

Issues:
  - Not uniform distance (1° latitude ≠ 1° longitude distance)
  - Euclidean metric breaks near poles
  - Not suitable for Euclidean heuristic
```

### Projected: UTM (EPSG:32648)

```
Hanoi uses UTM Zone 48N (meters)
Transformation:
  (lat, lon) in WGS84 → (x, y) in meters UTM

Example:
  (21.0285°N, 105.8523°E) → (481,234.5, 2,327,456.2) meters

Benefits:
  - Linear distance = √((x₁-x₂)² + (y₁-y₂)²)
  - Euclidean distance is valid heuristic
  - Straight-line distance ≤ road distance (admissible)
```

### Conversion Process

```
OSM graph (lat/lon) 
    ↓ [pyproj.Transformer]
UTM-projected graph (meters)
    ↓ [Euclidean distance on UTM]
Valid A* heuristic
```

---

## Performance Tuning Opportunities

### For CPU Code

1. **SIMD vectorization**: Process multiple nodes in parallel using AVX-512
2. **Cache blocking**: Organize graph to improve L3 cache locality
3. **Open list optimization**: Use fibonacci heap instead of binary heap

### For GPU Code

1. **Warp-level parallelism**: Exploit shared memory for open list management
2. **Memory coalescing**: Ensure all warps access aligned addresses
3. **Occupancy**: Keep registers-per-thread low to increase block count
4. **Shared memory**: Use for g_score cache to reduce global memory traffic

---

## Known Limitations

### Algorithm

1. **Directed graph**: Treats all edges as one-way
   - Real-world streets can have bidirectional traffic
   - Current approach: only follows actual directed edges
   - Limitation: May miss valid routes in some scenarios

2. **Euclidean heuristic**: Not perfect
   - Real road networks have barriers, one-way streets, etc.
   - Better heuristics: Landmark-based, contraction hierarchies
   - Tradeoff: Computation vs. search efficiency

### Implementation

1. **No dynamic weights**: Graph is static (no real-time traffic)
2. **Integer node IDs only**: No support for node names
3. **Uniform heap operations**: No specialized heap for GPU

---

## References

- A* Search Algorithm: Hart, Nilsson, Raphael (1968)
- CSR Format: https://en.wikipedia.org/wiki/Sparse_matrix
- CUDA Best Practices: NVIDIA CUDA Programming Guide
- OSMnx Documentation: https://osmnx.readthedocs.io/
- UTM Projection: https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system

---

**Last Updated**: June 2026
