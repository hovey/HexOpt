# HexOpt

Fork of [CMU-CBML/HexOpt](https://github.com/CMU-CBML/HexOpt) — a C++
program for optimizing all-hexahedral mesh quality (Scaled Jacobian
gradient optimization with Laplacian smoothing), associated with Hua
Tong's HexOpt papers.

## Build (macOS)

The upstream project is set up for Windows/MSVC
(`HexOpt.sln`/`HexOpt.vcxproj`). This fork adds a `CMakeLists.txt` for
building on macOS with Apple Clang + Homebrew's `libomp`.

```bash
brew install libomp
cmake -S . -B build
cmake --build build
cd .
./build/HexOpt
```

`optimizer.cpp`'s `main()` hardcodes relative input filenames
(`isidore_horse.obj`, `isidore_horse.vtk`, `mid2Fem.txt`), so the binary
must be run from the repo root, not from `build/`. It runs as an infinite
optimization loop, printing progress every 1000 iterations and writing
output at each Scaled-Jacobian threshold step.

## Update log

### 2026-07-03

- Identified that `meshQuality.cpp`/`meshQuality.h` — containing the core
  optimization algorithm (`lapSmth`, `sJGrad`, gradient routines) — had
  been deleted upstream in commit `8ecb9ac` ("ClearAllContent"),
  breaking the build. Emailed Hua Tong to request the files be restored
  upstream rather than recovering them unilaterally from git history.
- Hua Tong restored `meshQuality.cpp`/`.h` upstream
  (`CMU-CBML/HexOpt@5f46bf4`). Added `upstream` remote, fast-forwarded
  this fork's `main` onto it, and pushed to `hovey/HexOpt`.
- Added `CMakeLists.txt` for a macOS build: locates Homebrew's `libomp`
  via `brew --prefix libomp` and links `OpenMP::OpenMP_CXX`, since Apple
  Clang has no built-in OpenMP support.
- Installed `libomp` via Homebrew.
- Fixed a portability bug in `meshQuality.h`: `DBL_MAX` is used without
  including `<cfloat>`. MSVC pulls it in transitively through other
  headers; Apple Clang does not. Added the explicit include.
- Confirmed the project builds cleanly on macOS (Apple Clang 21,
  arm64) with only harmless `-Wformat` warnings in `optimizer.cpp`, and
  produced a working `build/HexOpt` binary.
