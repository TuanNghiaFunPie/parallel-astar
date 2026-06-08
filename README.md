# Parallel A* Pathfinding on GPU using CUDA

A high-performance implementation of the A* pathfinding algorithm optimized for GPU execution using CUDA, compared against sequential CPU implementations. This project demonstrates parallel computing techniques applied to real-world routing problems on the Hanoi street network.

## 📋 Project Overview

This project implements and benchmarks:
- **CPU Sequential A***: Single-threaded baseline in Python
- **GPU Parallel A***: Task-level parallelism with multiple threads per block

**Dataset**: OpenStreetMap road network of Hanoi (~45,000 nodes, ~120,000 edges)
- Coverage: Hoàn Kiếm and Ba Đình districts
- Graph type: Directed network (preserves one-way streets)
- Graph Format: Compressed Sparse Row (CSR) - optimized for sparse graph storage and GPU memory efficiency

---

## 🎯 Key Features

✅ **GPU Acceleration**: CUDA implementation for massive parallelism  
✅ **Real-world Data**: OSM street network with realistic pathfinding queries  
✅ **Benchmark Suite**: Scalability analysis with 10 and 1,000 query batches  
✅ **Visualization**: Interactive maps showing query locations and optimal paths  
✅ **Cross-Platform**: Works on Windows, Linux, macOS with relative paths

---

## 🔧 Prerequisites & Installation

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
   ```bash
   python --version  # Check version
   ```
   Download: https://www.python.org/downloads/

2. **CUDA Toolkit 11.0+** (for GPU support)
   ```bash
   nvcc --version  # Check CUDA version
   ```
   Download: https://developer.nvidia.com/cuda-downloads

3. **C++ Compiler**
   - Windows: Visual Studio Build Tools (included with CUDA)
   - Linux: GCC (`sudo apt-get install build-essential`)
   - macOS: Xcode Command Line Tools (`xcode-select --install`)

### Installation Steps

**1. Clone or download the project:**
```bash
git clone https://github.com/your-username/parallel-astar.git
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
You should see `(venv)` prefix in your terminal.

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

**5. Verify setup:**
```bash
python -c "import numpy, osmnx, folium; print('All imports OK!')"
nvcc --version  # Verify CUDA Toolkit
```

---

## 🚀 Quick Start - Running Benchmarks

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

**CPU Sequential (10 queries):**
```bash
python script/cpu_seq_10.py
```

**CPU Sequential (1,000 queries):**
```bash
python script/cpu_seq_1000.py
```

**GPU Parallel (10 queries):**
```bash
cd script
nvcc -O2 gpu_par_block_threads_10.cu -o gpu_par_block_threads_10
./gpu_par_block_threads_10
```

**GPU Parallel (1,000 queries):**
```bash
nvcc -O2 gpu_par_block_threads_1000.cu -o gpu_par_block_threads_1000
./gpu_par_block_threads_1000
```

### Visualization

Generate interactive maps showing query routes:
```bash
python script/04_visualize_map.py           # Full map + all routes
python script/05_visualize_10_queries.py    # Map with 10 queries
python script/06_visualize_1000_queries.py  # Map with 1000 queries
```

---

## 📁 Project Structure

```
parallel-astar/
├── script/                    # Main source code
│   ├── 01_download_osm.py    # Download Hanoi street network from OSM
│   ├── 02_build_csr.py       # Convert graph to CSR format
│   ├── 03_generate_queries.py # Generate pathfinding queries
│   ├── 04-06_visualize*.py   # Visualization scripts
│   ├── cpu_seq_10.py         # CPU baseline (10 queries)
│   ├── cpu_seq_1000.py       # CPU baseline (1000 queries)
│   ├── gpu_par_block_threads_10.cu    # GPU implementation (10 queries)
│   ├── gpu_par_block_threads_1000.cu  # GPU implementation (1000 queries)
│   ├── compile_and_run.sh    # Build script (Linux/macOS)
│   └── compile_and_run.bat   # Build script (Windows)
├── data/                     # OSM graph & query data
│   ├── hanoi_hoankiem_badinh.graphml  # Original OSM graph
│   ├── graph_csr.npz         # CSR format (row_ptr, col_index, data)
│   ├── queries_10.txt        # 10 landmark queries
│   ├── queries_1000.txt      # 1000 random queries
│   └── *.html                # Interactive map visualizations
├── cache/                    # Internal cache (query results)
├── requirements.txt          # Python dependencies
├── README.md                 # This file
├── SETUP.md                  # Detailed setup (for reference)
├── IMPLEMENTATION_NOTES.md   # Technical deep dive
├── CONTRIBUTING.md           # Contribution guidelines
├── LICENSE                   # MIT License
└── .gitignore               # Git ignore patterns
```

**Separation of Concerns:**
- `script/` - All execution logic (data processing, algorithms, visualization)
- `data/` - Input datasets and generated results
- `cache/` - Intermediate computation results
- Root directory - Configuration, documentation, metadata

---

## 📚 Documentation

- **[IMPLEMENTATION_NOTES.md](IMPLEMENTATION_NOTES.md)** - Technical architecture, algorithm details, GPU optimization strategies
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - How to contribute, code style, PR guidelines

---

## ⚙️ Troubleshooting

### Python Import Errors
```bash
# Ensure virtual environment is activated
source venv/bin/activate  # Linux/macOS
venv\Scripts\activate     # Windows

# Reinstall dependencies
pip install --upgrade -r requirements.txt
```

### CUDA Not Found
```bash
# Check CUDA installation
nvcc --version

# If not found, install CUDA Toolkit from:
# https://developer.nvidia.com/cuda-downloads
```

### Build Errors on Windows
- Ensure Visual Studio Build Tools are installed
- Check that CUDA Toolkit matches your Visual Studio version
- Run in **Developer Command Prompt**

### Permission Denied (Linux/macOS)
```bash
chmod +x compile_and_run.sh
./compile_and_run.sh all
```

---

## 📊 Benchmark Results

Expected performance (on NVIDIA GTX 1080):
- **CPU (10 queries)**: ~150-200 ms
- **GPU (10 queries)**: ~50-80 ms
- **GPU (1000 queries)**: ~400-600 ms (batched)

See output files in `data/` after running benchmarks.

---

## 📖 FAQ

**Q: Why CSR format?**  
A: Road networks are sparse. CSR saves ~98% memory vs dense matrix (0.66 MB vs 8 GB).

**Q: Can I run without GPU?**  
A: Yes, CPU scripts run on any Python environment. GPU scripts require NVIDIA GPU + CUDA.

**Q: How do I add new queries?**  
A: Edit `script/03_generate_queries.py` to change query generation logic, then rerun.

**Q: Why are results different each run?**  
A: GPU floating-point operations have minor precision variations; CPU is deterministic.

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

## 🤝 Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

**Questions?** Open an issue or check [IMPLEMENTATION_NOTES.md](IMPLEMENTATION_NOTES.md) for technical details.
```

**Mở kết quả:** Các file `.html` được lưu trong folder `data/`
- `data/area_map.html`
- `data/map_visualization_10_queries.html`
- `data/map_visualization_1000_queries.html`

---

## 📁 Project Structure

```
parallel-astar/
├── README.md                       # Main documentation
├── SETUP.md                        # Installation guide
├── IMPLEMENTATION_NOTES.md         # Technical deep-dive
├── CONTRIBUTING.md                # Contribution guidelines
├── LICENSE                         # MIT License
├── requirements.txt                # Python dependencies
├── .gitignore                      # Git ignore rules
│
├── compile_and_run.sh             # Build script (Linux/macOS)
├── compile_and_run.bat            # Build script (Windows)
│
├── script/                        # All source code
│   ├── 01_download_osm.py         # Download OSM graph
│   ├── 02_build_csr.py            # Convert to CSR format
│   ├── 03_generate_queries.py     # Generate test queries
│   ├── 04_visualize_map.py        # Visualization helper
│   ├── 05_visualize_10_queries.py # Visualize 10 queries
│   ├── 06_visualize_1000_queries.py # Visualize 1000 queries
│   ├── cpu_seq_10.py              # CPU baseline (10 queries)
│   ├── cpu_seq_1000.py            # CPU baseline (1000 queries)
│   ├── gpu_par_block_threads_10.cu  # GPU implementation (10 queries)
│   └── gpu_par_block_threads_1000.cu # GPU implementation (1000 queries)
│
├── data/                          # Output data (generated)
│   ├── hanoi_hoankiem_badinh.graphml   # OSM graph
│   ├── graph_csr.npz              # CSR format
│   ├── coords.txt                 # Node coordinates
│   ├── queries_10.txt             # 10 landmark queries
│   ├── queries_1000.txt           # 1000 random queries
│   └── *.html                     # Visualization maps
│
└── cache/                         # Nominatim cache
    └── (OSM query results)
```

---

## 🔧 Algorithm Details

### Thuật toán A* (A* Pathfinding)

**Công thức:** $f(n) = g(n) + h(n)$

- $g(n)$: Chi phí thực tế từ điểm bắt đầu đến nút $n$
- $h(n)$: Ước lượng heuristic từ $n$ đến đích (khoảng cách Euclidean trên tọa độ UTM)
- $f(n)$: Tổng chi phí ước tính

**Cơ chế hoạt động:**
- Duy trì **Open Set**: Hàng đợi ưu tiên (min-heap) sắp xếp theo f-score
- Duy trì **Closed Set**: Tập hợp các nút đã đánh giá
- Luôn mở rộng nút có f-score nhỏ nhất
- Dừng khi đạt đến đích

**Tính chất Admissible:**
Vì $h(n)$ là khoảng cách Euclidean (không bao giờ đánh giá quá cao chi phí thực tế), nên A* luôn tìm ra đường đi tối ưu.

### Biểu diễn Đồ thị: Compressed Sparse Row (CSR)

**Tại sao CSR?**
- **Tiết kiệm bộ nhớ:** $O(N + E)$ so với $O(N^2)$ của ma trận kề
  - Dày đặc: 45K × 45K = **2.025 tỷ ô** ≈ **8GB** (không khả thi)
  - CSR: ~45K + 120K = **165K ô** ≈ **0.66MB** (khả thi)
- **Truy cập bộ nhớ GPU tối ưu (Coalescing):** Dữ liệu liên tục → cache tốt
- **Tìm kiếm lân cận nhanh:** `neighbors(u) = col_index[row_ptr[u]:row_ptr[u+1]]`

**Cấu trúc CSR:**
```
row_ptr[i]     = chỉ số bắt đầu trong col_index cho các lân cận của nút i
col_index[]    = ID nút đích (kích thước = số cạnh)
edge_weight[]  = trọng số cạnh/chiều dài (kích thước = số cạnh)
```

**Ví dụ:**
```
Đồ thị: 0→1(w=5), 0→2(w=3), 2→1(w=2)

CSR:
  row_ptr   = [0, 2, 2, 3]
  col_index = [1, 2, 1]
  weights   = [5, 3, 2]

Lân cận(0) = col_index[0:2] = {1, 2}, trọng số = {5, 3}
```

### Chiến lược Song song hóa trên GPU

**Task-level Parallelism:** Một block xử lý một truy vấn
- **Cấp độ song song:** Tất cả các truy vấn (1000 blocks chạy song song cho 1000 queries)
- **Trong mỗi block:** Nhiều threads ($T = 128$ hoặc 256 threads) hợp tác xử lý cùng một truy vấn A*

**Chi tiết implement:**

```cuda
// Một block = một query
__global__ void astarKernel(...) {
    int qid = blockIdx.x;  // Query ID từ block
    
    // Threads trong block hợp tác:
    // - Thread 0: Quản lý open_set (min-heap)
    // - Threads 0..T-1: Khởi tạo song song, cập nhật g_score
    // - __syncthreads(): Đồng bộ hóa
}
```

**Shared Memory Usage:**
```
- s_heapSize: Kích thước open_set (heap)
- s_expanded: Số lượng nút đã mở rộng
- s_maxQ: Kích thước tối đa của queue
- s_to_push[64]: Nút cần thêm vào heap
```

**Ưu điểm:**
- ✓ Nhiều queries chạy song song (throughput cao)
- ✓ Threads trong block hợp tác → khởi tạo nhanh
- ✓ Shared memory chia sẻ dữ liệu hiệu quả

**Tradeoff:**
- Một block chỉ xử lý được 1 query (không song song hóa trong query)
- Nếu query nhỏ, một số threads sẽ không làm việc

---

## 📊 Kết quả Benchmark

### Các chỉ số Đo lường

Mỗi benchmark thu thập:
1. **Thời gian thực hiện (ms)**: Tổng thời gian cho tất cả queries
2. **Số nút mở rộng**: Số nút được A* đánh giá (chỉ số hiệu quả thuật toán)
3. **Speedup**: Thời gian CPU / Thời gian GPU
4. **Throughput**: Queries per second

### Kỳ vọng Hiệu năng

| Triển khai | Batch | Thời gian (ms) | Nút mở rộng | Speedup |
|---|---|---|---|---|
| CPU tuần tự | 10 | ~100-300 | ~500-1000/query | 1.0× |
| GPU parallel | 10 | ~30-100 | ~500-1000/query | 2-5× |
| CPU tuần tự | 1000 | ~10,000-30,000 | ~500-1000/query | 1.0× |
| GPU parallel | 1000 | ~1,000-5,000 | ~500-1000/query | 5-15× |

**Lưu ý:** Thời gian thực tế phụ thuộc vào GPU, CPU, độ phức tạp query, và số threads/block.

**Cách chạy benchmark:**
```bash
# 10 queries
python script/cpu_seq_10.py
cd script && ./gpu_par_block_threads_10

# 1000 queries
python script/cpu_seq_1000.py
cd script && ./gpu_par_block_threads_1000
```

---

## 💡 Chi tiết Triển khai

### Tối ưu hóa Bộ nhớ trên GPU

**Shared Memory (mỗi block):**
```cuda
__shared__ int s_heapSize;     // Kích thước open_set
__shared__ int s_expanded;     // Nút đã mở rộng
__shared__ Node s_to_push[64]; // Buffer thêm nút vào heap
```

**Global Memory:**
```cuda
float* g         // g_score array (V phần tử)
int* p           // parent array (V phần tử)
Node* heap       // open_set heap (V phần tử)
bool* closed     // closed set (V phần tử)
```

**Registers:** Loop counters, temporary values

### Chiến lược Đồng bộ hóa

- **Block-level:** `__syncthreads()` để threads hợp tác
- **Query-level:** Không cần đồng bộ giữa blocks (query độc lập)

### Khởi tạo Hiệu quả

```cuda
// Threads chia sẻ khởi tạo arrays
for(int i=threadIdx.x; i<V; i+=blockDim.x){
    g[i] = INFINITY;
    p[i] = -1;
    closed[i] = false;
}
__syncthreads();  // Đợi tất cả threads
```

### Quản lý Open Set (Min-Heap)

Thread 0 chính quản lý heap:
- **Push:** Thêm nút mới vào heap (bubble-up)
- **Pop:** Lấy nút có f-score nhỏ nhất (heapify-down)
- **Thread khác:** Chỉ đọc kết quả thông qua shared memory

### Hàm Heuristic

- **Khoảng cách Euclidean:** Tính trên tọa độ UTM (mét)
- **Admissible:** Không bao giờ đánh giá quá cao chi phí thực tế
- **Consistent:** $h(n) \leq \text{cost}(n \to m) + h(m)$ cho mọi cạnh

**Công thức:**
$$h(n) = \sqrt{(x_n - x_{\text{goal}})^2 + (y_n - y_{\text{goal}})^2}$$

Cách tính trên GPU (CUDA):
```cuda
float heuristic(int u, int goal, const float* coords) {
    float dx = coords[2*u] - coords[2*goal];
    float dy = coords[2*u+1] - coords[2*goal+1];
    return sqrt(dx*dx + dy*dy);
}
```

---

## 📈 Phân tích Scaling

**10 queries:**
- Kiểm tra tính đúng của thuật toán
- Overhead kernel launch có ảnh hưởng

**1000 queries:**
- Cho thấy lợi ích song song hóa
- GPU hoàn toàn chiếm ưu thế
- Amortized kernel overhead

**Xu hướng:** GPU tốt hơn CPU khi số lượng queries tăng.

---

## 🛠️ Biên dịch từ Source

### Windows
```bash
cd script
nvcc -O2 gpu_par_block_threads_10.cu -o gpu_par_block_threads_10.exe
gpu_par_block_threads_10.exe
```

### Linux
```bash
cd script
nvcc -O2 gpu_par_block_threads_10.cu -o gpu_par_block_threads_10
./gpu_par_block_threads_10
```

---

## 📚 Công nghệ Sử dụng

- **CUDA C++**: GPU programming, parallel algorithms
- **Python**: Data preprocessing, CPU baseline
- **OSMnx**: OpenStreetMap data integration
- **NetworkX**: Graph data structures
- **NumPy**: Array processing
- **Folium**: Interactive mapping
- **Pyproj**: Coordinate transformation (lat/lon ↔ UTM)

---

## 📄 Nguồn Dữ liệu

- **Mạng đường Hà Nội:** OpenStreetMap (qua thư viện OSMnx)
- **Phạm vi:** Quận Hoàn Kiếm và Ba Đình
- **Loại mạng:** Các đường xe ô tô (drive)
- **Số nút:** ~45,000 | **Số cạnh:** ~120,000

---

## 📝 Giấy phép

MIT License - Xem file LICENSE để chi tiết

---

## 👥 Tác giả

- Bùi Xuân Vinh (23110232)
- Nguyễn Tuấn Nghĩa (23110204)

**Giảng viên:** TS. Lê Kim Quy  
**Môn học:** Tính toán Song song  
**Trường:** [Your University]

---

## 🔗 Tài liệu Tham khảo

- [A* Search Algorithm - Wikipedia](https://en.wikipedia.org/wiki/A*_search_algorithm)
- [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)
- [CSR Format - Wikipedia](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_row_(CSR,_CRS_or_Yale_format))
- [OSMnx Documentation](https://osmnx.readthedocs.io/)
- [UTM Coordinate System](https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system)

---

## ❓ Câu hỏi Thường gặp

**Q: Tại sao GPU chậm hơn CPU với 10 queries?**  
A: Overhead khởi tạo kernel. GPU tỏa sáng khi queries > 100+ khi amortized overhead.

**Q: Tôi có thể dùng thành phố khác không?**  
A: Có! Sửa `script/01_download_osm.py` để chỉ định khu vực khác via `ox.geocode_to_gdf()`.

**Q: Tại sao CSR thay vì ma trận kề?**  
A: CSR tiết kiệm 99.9% bộ nhớ cho đồ thị thưa + GPU coalesced access nhanh hơn.

**Q: Số threads per block tối ưu là bao nhiêu?**  
A: Thường 128 hoặc 256 threads. Phụ thuộc GPU, register pressure, shared memory.

**Q: Làm sao để profile GPU code?**  
A: Dùng NVIDIA Nsight Systems:
```bash
nsys profile ./script/gpu_par_block_threads_10
```

**Q: Kết quả đường đi có khác nhau giữa CPU và GPU không?**  
A: Không - cả hai cùng implement A*, chỉ khác phần cứng thực thi.

---

**Cập nhật cuối:** Tháng 6, 2026  
**Trạng thái:** ✅ Sẵn sàng cho Portfolio / GitHub


---

# Phần bổ sung từ README(1).md

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
