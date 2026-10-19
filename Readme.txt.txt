Parallel Split-Step Fourier Method — 1D NLSE Solver
Course: ID5130 Parallel Scientific Computing, IIT Madras

FOLDER STRUCTURE

Codes/
├── Benchmark/
│   ├── serial_benchmark.cpp
│   ├── openmp_benchmark.cpp
│   ├── openacc_benchmark.cpp
│   ├── fft_serial_benchmark.cpp
│   ├── cufft_benchmark.cpp
│   ├── plot_benchmarks.py
│   └── plot_fft_benchmarks.py
│
├── Validation/
│   ├── serial_val.cpp
│   ├── openmp_val.cpp
│   ├── openacc_val.cpp
│   ├── fft_val.cpp
│   ├── cufft_val.cpp
│   └── compare_solutions.py
│
├── nlse_dynamics.cpp
└── plot_fiber_dynamics.py

DEPENDENCIES
  - g++      : for serial and OpenMP builds
  - nvc++    : NVIDIA HPC SDK compiler, for OpenACC and cuFFT builds
  - FFTW3    : for fft_serial_benchmark  (link with -lfftw3)
  - cuFFT    : for cufft_benchmark       (via nvc++ -cudalib=cufft)
  - Python 3 : numpy, pandas, matplotlib

BENCHMARK FOLDER
The Benchmark folder is fully automated. Running either Python script will
compile all relevant C++ files, execute them, and save the output plot.

  Naive DFT benchmarks (Serial, OpenMP, OpenACC):
      cd Codes/Benchmark
      python3 plot_benchmarks.py

  Output: benchmark_results.png

  FFT library benchmarks (FFTW3 vs cuFFT):
      cd Codes/Benchmark
      python3 plot_fft_benchmarks.py

  Output: fft_benchmark_results.png

  Each benchmark sweeps grid sizes N = 512 to 8192 (naive DFT) and
  N = 512 to 65536 (FFT library), running 100 SSFM steps per size
  repeated 3 times. Raw terminal output is saved as <name>_out.txt.

VALIDATION FOLDER

The validation C++ files must be compiled and run manually before
running the Python comparison script.

  Step 1 — Compile:

      g++ -O3 serial_val.cpp -o serial_val
      g++ -O3 -fopenmp openmp_val.cpp -o openmp_val
      nvc++ -acc -fast openacc_val.cpp -o openacc_val
      g++ -O3 fft_val.cpp -o fft_val -lfftw3
      nvc++ -acc -fast -cudalib=cufft cufft_val.cpp -o cufft_val

  Step 2 — Run each:

      ./serial_val        # writes serial_out.csv
      ./openmp_val        # writes openmp_out.csv
      ./openacc_val       # writes openacc_out.csv
      ./fft_val           # writes fft_out.csv
      ./cufft_val         # writes cufft_out.csv

  Step 3 — Run the comparison script:

      python3 compare_solutions.py

  This computes the L-infinity norm error of each parallel implementation
  against the serial result, prints a PASS/FAIL summary, and saves
  solution_comparison.png. Note that the FFT and cuFFT comparisons are
  commented out in the script by default and can be enabled if needed.

NLSE DYNAMICS (ROOT LEVEL)

nlse_dynamics.cpp simulates a Gaussian pulse in a strongly nonlinear fiber
(gamma = 5.0) using FFTW3, writing the full spatio-temporal evolution to
wavefunction_dynamics.csv. plot_fiber_dynamics.py then reads this CSV and
generates the dynamics plots.

  Step 1 — Compile:
      g++ -O3 nlse_dynamics.cpp -o nlse_dynamics -lfftw3

  Step 2 — Run:
      ./nlse_dynamics

  Step 3 — Plot:
      python3 plot_fiber_dynamics.py