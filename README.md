```markdown
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

```

**2. Create Python virtual environment:**

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/macOS
python3 -m venv venv
source venv/bin/activate

```

**3. Install Python dependencies:**

```bash
pip install --upgrade pip
pip install -r requirements.txt

```

**4. Prepare data (one-time setup):**

```bash
python script/01_download_osm.py        # Download Hanoi street network
python script/02_build_csr.py           # Convert to CSR format
python script/03_generate_queries.py    # Generate pathfinding queries

```

---

## 🚀 Quick Start / Hướng dẫn Chạy

### Method 1: Using Build Scripts (Recommended)

```bash
# Linux/macOS
./compile_and_run.sh cpu_10      # CPU benchmark (10 queries)
./compile_and_run.sh gpu_10      # GPU benchmark (10 queries)
./compile_and_run.sh gpu_1000    # GPU benchmark (1000 queries)
./compile_and_run.sh all         # Run all benchmarks

# Windows
compile_and_run.bat cpu_10       # CPU benchmark
compile_and_run.bat gpu_10       # GPU benchmark
compile_and_run.bat gpu_1000     # GPU benchmark (1000 queries)
compile_and_run.bat all          # Run all benchmarks

```

### Method 2: Direct Execution

**CPU Sequential:**

```bash
python script/cpu_seq_10.py
python script/cpu_seq_1000.py

```

**GPU Parallel (128 threads/block):**

```bash
cd script
nvcc -O2 gpu_par_block_threads_10.cu -o gpu_par_block_threads_10
./gpu_par_block_threads_10

nvcc -O2 gpu_par_block_threads_1000.cu -o gpu_par_block_threads_1000
./gpu_par_block_threads_1000

```

### Visualization / Trực quan hóa

Các file kết quả `.html` được lưu trong folder `data/` sau khi chạy:

```bash
python script/04_visualize_map.py           # data/area_map.html
python script/05_visualize_10_queries.py    # data/map_visualization_10_queries.html
python script/06_visualize_1000_queries.py  # data/map_visualization_1000_queries.html

```

---

## 📁 Project Structure / Cấu trúc Thư mục

```text
parallel-astar/
├── README.md                       # Main documentation
├── SETUP.md                        # Installation guide
├── IMPLEMENTATION_NOTES.md         # Technical deep-dive
├── CONTRIBUTING.md                 # Contribution guidelines
├── LICENSE                         # MIT License
├── requirements.txt                # Python dependencies
├── .gitignore                      # Git ignore rules
├── compile_and_run.sh              # Build script (Linux/macOS)
├── compile_and_run.bat             # Build script (Windows)
│
├── script/                         # All source code (Python & CUDA)
│   ├── 01_download_osm.py         
│   ├── 02_build_csr.py            
│   ├── 03_generate_queries.py     
│   ├── 04_visualize_map.py        
│   ├── 05_visualize_10_queries.py 
│   ├── 06_visualize_1000_queries.py 
│   ├── cpu_seq_10.py              
│   ├── cpu_seq_1000.py            
│   ├── gpu_par_block_threads_10.cu  
│   └── gpu_par_block_threads_1000.cu 
│
├── data/                           # Output data (generated)
│   ├── hanoi_hoankiem_badinh.graphml   
│   ├── graph_csr.npz              
│   ├── coords.txt                 
│   ├── queries_10.txt             
│   ├── queries_1000.txt           
│   └── *.html                     
│
└── cache/                          # Nominatim cache
    └── (OSM query results)

```

---

## 🔧 Algorithm Details / Chi tiết Thuật toán

### Thuật toán A* (A* Pathfinding)

**Công thức:** $f(n) = g(n) + h(n)$

* $g(n)$: Chi phí thực tế từ điểm bắt đầu đến nút $n$
* $h(n)$: Ước lượng heuristic từ $n$ đến đích (khoảng cách Euclidean trên tọa độ UTM)
* $f(n)$: Tổng chi phí ước tính

### Biểu diễn Đồ thị: Compressed Sparse Row (CSR)

* **Tiết kiệm bộ nhớ:** $O(N + E)$ thay vì ma trận kề $O(N^2)$. Giảm từ ~8GB xuống ~0.66MB.
* **Truy cập bộ nhớ GPU tối ưu (Coalescing):** Dữ liệu liên tục giúp tận dụng cache.

### Chiến lược Song song hóa trên GPU

**Task-level Parallelism:** Một block xử lý một truy vấn, sử dụng 128 threads/block.

* **Cấp độ song song:** Tất cả các truy vấn chạy song song trên các block độc lập.
* **Trong mỗi block:** 128 threads hợp tác xử lý khởi tạo mảng và cập nhật lân cận (g_score). Thread 0 quản lý hàng đợi ưu tiên (min-heap).

---

## 📊 Benchmark Results / Kết quả Đánh giá

| Triển khai | Batch | Thời gian (ms) | Speedup |
| --- | --- | --- | --- |
| CPU tuần tự | 10 | ~100-300 | 1.0× |
| GPU parallel (128 threads/block) | 10 | ~30-100 | 2-5× |
| CPU tuần tự | 1000 | ~10,000-30,000 | 1.0× |
| GPU parallel (128 threads/block) | 1000 | ~1,000-5,000 | 5-15× |

*Đo lường thực tế phụ thuộc vào cấu hình phần cứng.*

---

## 📚 Công nghệ Sử dụng & Tài liệu Tham khảo

* **Ngôn ngữ & Thư viện:** CUDA C++, Python, OSMnx, NetworkX, NumPy, Folium.
* [A* Search Algorithm - Wikipedia](https://en.wikipedia.org/wiki/A*_search_algorithm)
* [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)
* [CSR Format - Wikipedia](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_row_(CSR,_CRS_or_Yale_format))
