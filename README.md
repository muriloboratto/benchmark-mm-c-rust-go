# Matrix Multiplication Benchmark: C++ vs. Rust vs. Go

A reproducible, educational Jupyter Notebook for comparing **single-threaded dense matrix multiplication** implemented in **C++**, **Rust**, and **Go**. The experiment uses the same numerical algorithm and input data in all three languages, validates the computed results, and compares **median kernel execution time** and **speedup relative to C++**.

## Notebook

[Google Colab](https://colab.research.google.com/drive/1V9UT34JfXH7lQmQirWipBj3LHyUlUEhZ?usp=sharing)

## Requirements

- Ubuntu or Debian Linux, with network access and `sudo` privileges for compiler installation (or run as root without `sudo`).
- Jupyter Notebook or JupyterLab with a Python kernel.
- Python packages: `pandas` and `matplotlib`.

Install the compilers from a terminal or the notebook's installation cell:

```bash
sudo apt-get update
sudo apt-get install -y build-essential rustc cargo golang-go
```

Install the Python dependencies if they are not already available:

```bash
python3 -m pip install jupyter pandas matplotlib
```

> Depending on your distribution's Python packaging policy, use a virtual environment for the Python dependencies. The notebook itself contains `apt-get install` but does not run `apt-get update` or install the Python packages.

## Run the experiment

1. Place the notebook in a working directory and launch `jupyter lab` (or `jupyter notebook`).
2. Open `benchmark_cpp_rust_go_english(1).ipynb` and run the cells in order.
3. The notebook writes source files to `matrix_benchmark/`, compiles optimized executables, verifies numerical correctness, and collects timings.
4. Review the execution-time table and the two generated plots.

The notebook generates these files locally:

```text
matrix_benchmark/
├── mm.cpp
├── mm.rs
├── mm.go
├── mm_cpp
├── mm_rust
└── mm_go
```

## Benchmark methodology

All implementations use square, row-major `float64` matrices and the same sequential **i–k–j** loop ordering:

\[
C_{ij} = \sum_{k=0}^{N-1} A_{ik} B_{kj}
\]

The input matrices are initialized deterministically:

- `A[i,k] = ((i+k) % 7) + 1`
- `B[k,j] = ((k+j) % 5) + 1`
- `C` starts with zero values.

The default matrix sizes are **128, 256, and 512**. Each executable runs **one warm-up** followed by **five measured runs** per size. The notebook reports the **median elapsed time in seconds**. Timing is performed inside the compiled program around the multiplication kernel, excluding process startup, matrix initialization, and checksum calculation.

### Compilation settings

| Language | Build command / optimization |
| --- | --- |
| C++ | `g++ -O3 -std=c++17` |
| Rust | `rustc -C opt-level=3` |
| Go | `go build` (default optimized build) |

These settings are not strictly equivalent between toolchains.

### Correctness verification

Each executable prints its measured execution time and a checksum (sum of all elements of `C`). Python compares the checksum with an independently computed expected value using `math.isclose` (`rel_tol=1e-10`, `abs_tol=1e-7`). A failed check stops the benchmark.

## Outputs

The notebook displays a table with one row per matrix size and one execution-time column per language:

| Matrix Size | C++ (s) | Rust (s) | Go (s) |
| --- | ---: | ---: | ---: |
| 128 | Measured | Measured | Measured |
| 256 | Measured | Measured | Measured |
| 512 | Measured | Measured | Measured |

It also produces two line charts:

1. **Median Kernel Execution Time** — lower values indicate faster execution.
2. **Speedup Relative to C++** — computed as `T_C++ / T_language`. C++ is the **1.0× baseline**; values above 1.0× indicate faster execution than C++, and values below 1.0× indicate slower execution.

The current notebook does **not** generate a GFLOP/s chart.

## Customize

In the benchmark cell, modify:

```python
SIZES = [128, 256, 512]  # Add 1024 for a longer experiment
REPEATS = 5
```

Larger matrices increase runtime and memory requirements. For more reliable comparisons, use an otherwise idle machine, record CPU and compiler versions, and repeat the experiment under similar conditions.

## Limitations and possible extensions

- This benchmark measures a **single-threaded, naive matrix multiplication kernel**; it does not compare optimized BLAS libraries or GPU implementations.
- No explicit multithreading is used (e.g., OpenMP, Rayon, or goroutines).
- Different compilers may optimize the same source-level algorithm differently.
- Memory hierarchy, CPU frequency scaling, thermal conditions, and background tasks can influence timings.
- Potential next steps include cache blocking/tiling, SIMD analysis, multicore parallelism, and comparisons against BLAS implementations.

## License

No license is specified in the notebook. Add a license file if you intend to distribute or reuse this project under explicit terms.
