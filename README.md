# Low-Latency Monte Carlo Option Pricer

This project demonstrates the progressive optimization of a C++17 Monte Carlo simulation for pricing European call options. It follows a layered optimization approach, progressing from a single-threaded baseline to a multithreaded and SIMD-vectorized implementation optimized for x86-64 CPUs.

The implementation explores thread-level parallelism, AVX2 SIMD vectorization, vectorized mathematical functions using SLEEF, and an xoshiro128+ pseudorandom number generator.

## Core Concepts Demonstrated

- **Modern C++:** C++17, memory management, and performance-oriented programming.
- **Financial Modeling:** European call option pricing using Monte Carlo simulation and the Black–Scholes framework.
- **Concurrency:** Parallel execution using `std::thread` and independent workload partitioning.
- **CPU Optimization (SIMD):** AVX2 vectorization and FMA instructions for parallel floating-point operations.
- **Vectorized Mathematics:** SLEEF for vectorized mathematical functions.
- **Random Number Generation:** xoshiro128+ for fast pseudorandom number generation.
- **Performance Engineering:** Benchmarking and comparing progressively optimized implementations.

## Hardware and Software

- **CPU:** Intel Core i5-12500H (12th Gen)
- **Architecture:** x86-64
- **Operating System:** Linux
- **SIMD Instruction Set:** AVX2 (256-bit vectors)
- **Compiler:** GCC or Clang
- **Language Standard:** C++17
- **Math Library:** SLEEF
- **Parallelism:** C++ threads

## Project Structure

The project is organized into progressively optimized implementations.

- **`base/`**
  - `pricer.cpp`: Single-threaded baseline using scalar calculations and a straightforward simulation loop.

- **`threaded/`**
  - `pricer_threaded.cpp`: Distributes Monte Carlo paths among multiple threads while retaining scalar calculations.

- **`SIMD/`**
  - `pricer_avx2.cpp`: Uses AVX2 intrinsics and multithreading to process multiple simulation paths per vector operation.
  - `pricer_avx2_fully_vectorized.cpp`: Combines multithreading, SLEEF vector math, and the xoshiro128+ random number generator.

*Note: Adjust the filenames in the commands below to match the source files in your repository.*

## How to Build and Run

The following commands target an Intel/AMD x86-64 Linux system. The AVX2 implementations require a CPU that supports AVX2.

### 1. Baseline: Single-threaded

```bash
cd base
g++ -O3 -std=c++17 pricer.cpp -o pricer
./pricer
```

### 2. Multithreaded

```bash
cd threaded
g++ -O3 -std=c++17 -pthread \
    pricer_threaded.cpp -o pricer_threaded
./pricer_threaded
```

### 3. Multithreaded + AVX2 SIMD

```bash
cd SIMD
g++ -O3 -mavx2 -mfma -std=c++17 -pthread \
    pricer_avx2.cpp -o pricer_avx2
./pricer_avx2
```

### 4. Multithreaded + AVX2 + SLEEF

Install SLEEF using your Linux distribution's package manager or build it from source. Adjust the include and library paths if it is installed in a nonstandard location.

```bash
cd SIMD
g++ -O3 -mavx2 -mfma -std=c++17 -pthread \
    pricer_avx2_fully_vectorized.cpp \
    -lsleef -lm -o pricer_avx2_sleef
./pricer_avx2_sleef
```

The final command assumes that the source uses a compatible SLEEF vector API and that the library is installed in a discoverable location. Depending on the installation, additional include paths, library paths, or linker flags may be required.

## Performance Results

The benchmark evaluates the performance of the Monte Carlo option pricer across progressively optimized implementations on an Intel Core i5-12500H system.

The workload consists of 10,000,000 Monte Carlo simulation paths.

| Implementation | Runtime | Speedup vs. Baseline |
|---|---:|---:|
| Single-threaded baseline (`double`) | 0.205796 s | 1.00x |
| Multithreaded (`double`) | ~0.03056 s | ~6.73x |
| Multithreaded + AVX2 (`float`) | 0.012407 s | 16.60x |
| Multithreaded + SLEEF + AVX2 (`float`) | 0.00719 s | 28.60x |

Speedup is calculated by dividing the baseline runtime by the optimized runtime.

For a fair comparison, all implementations should use the same number of simulation paths, equivalent pricing parameters, and consistent benchmarking conditions.

## Optimization Analysis

### Step 1: Multithreading

The simulation workload is divided among independent threads. Each thread processes its assigned Monte Carlo paths and accumulates a local payoff sum. Local accumulation minimizes contention on shared data and reduces synchronization overhead.

### Step 2: AVX2 SIMD Vectorization

AVX2 provides 256-bit vector registers. A single vector can hold eight 32-bit floating-point values or four 64-bit floating-point values.

This allows multiple simulation paths to perform arithmetic operations simultaneously, increasing computational throughput.

### Step 3: Vectorized Mathematical Functions

SLEEF provides vectorized implementations of mathematical functions used in the simulation, potentially reducing the cost of operations such as exponential and logarithmic calculations.

Using vector math alongside AVX2 can improve performance in computationally intensive parts of the pricing pipeline.

### Step 4: Efficient Random Number Generation

The xoshiro128+ pseudorandom number generator provides a lightweight alternative to more expensive general-purpose random-number generation approaches.

Maintaining independent generator state per thread avoids shared-state synchronization and supports parallel execution.

## Correctness and Benchmarking

- Validate the estimated option price against the Black–Scholes analytical price.
- Ensure random-number generation is independent across threads and does not introduce data races.
- Compare numerical accuracy when switching from `double` to `float`.
- Warm up each implementation and perform repeated timing runs.
- Use consistent simulation parameters and workload sizes.
- Record compiler versions, optimization flags, thread counts, and hardware details.
- Compare performance against the single-threaded baseline.

## Conclusion

This project demonstrates how thread-level parallelism, AVX2 SIMD vectorization, efficient random-number generation, and vectorized mathematical functions can be combined to accelerate Monte Carlo option pricing.

By progressively optimizing the computational pipeline, the project explores key techniques used in high-performance C++ programming and low-latency numerical computing.
