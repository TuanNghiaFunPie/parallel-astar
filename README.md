# Parallel A* Pathfinding on GPU using CUDA

A high-performance implementation of the A* pathfinding algorithm optimized for GPU execution using CUDA, compared against sequential CPU implementations. This project demonstrates parallel computing techniques applied to real-world routing problems on the Hanoi street network.

## 📋 Project Overview / Tổng quan Dự án

This project implements and benchmarks:
- **CPU Sequential A***: Single-threaded baseline in Python
- **GPU Parallel A***: Task-level parallelism with multiple threads per block (128 threads/block)

**Dataset**: OpenStreetMap road network of Hanoi (~45,000 nodes, ~120,000 edges)
- Coverage: Hoàn Kiếm and Ba Đình districts
- Graph type: Directed network (preserves one-way streets)
- Graph Format: Compressed Sparse Row (CSR) - optimized for sparse graph storage and GPU memory efficiency

---

## 🎯 Key Features / Tính năng Chính

✅ **GPU Acceleration**: CUDA implementation for massive parallelism  
✅ **Real-world Data**: OSM street network with realistic pathfinding queries  
✅ **Benchmark Suite**: Scalability analysis with 10 and 1,000 query batches  
✅ **Visualization**: Interactive maps showing query locations and optimal paths  
✅ **Cross-Platform**: Works on Windows, Linux, macOS with relative paths

---

## 🔧 Prerequisites & Installation / Cài đặt

### System Requirements

**Windows:**
- Windows 10/11
- 8GB RAM (16GB recommended)
- NVIDIA GPU (GTX 1050 or better)

**Linux/macOS:**
- Ubuntu 18.04+, CentOS 7+, or macOS 10.14+
- 8GB RAM (16GB recommended)
- NVIDIA GPU with CUDA support

### Required Software

1. **Python 3.8+**
2. **CUDA Toolkit 11.0+**
3. **C++ Compiler** (Visual Studio Build Tools / GCC / Xcode)

### Installation Steps

**1. Clone or download the project:**
```bash
git clone [https://github.com/TuanNghiaFunPie/parallel-astar.git](https://github.com/TuanNghiaFunPie/parallel-astar.git)
cd parallel-astar
