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

## Related papers

Local PDFs of the papers behind this codebase and its sibling projects,
with links to the live versions:

| Paper | Venue | Link |
|---|---|---|
| HybridOctree_Hex: Hybrid octree-based adaptive all-hexahedral mesh generation with Jacobian control | J. Comput. Sci. 78 (2024) 102278 | [doi.org/10.1016/j.jocs.2024.102278](https://doi.org/10.1016/j.jocs.2024.102278) |
| Fast and Robust Hexahedral Mesh Optimization via Augmented Lagrangian, L-BFGS, and Line Search | arXiv:2410.11656 (Dec 2024, v3) | [arxiv.org/abs/2410.11656](https://arxiv.org/abs/2410.11656) |
| Element-Saving Hexahedral 3-Refinement Templates | arXiv:2512.14862 (Jan 2026, v4) | [arxiv.org/abs/2512.14862](https://arxiv.org/abs/2512.14862) |
| HexOpt: Efficient and robust hexahedral mesh optimization using Rectified Hybrid Quadratic Jacobian and geometry-aware mapping | Computer-Aided Design 196 (2026) 104073 | [doi.org/10.1016/j.cad.2026.104073](https://doi.org/10.1016/j.cad.2026.104073) |
| Element-saving hexahedral 3-refinement templates | Comput. Aided Geom. Des. 127 (2026) 102550 | [doi.org/10.1016/j.cagd.2026.102550](https://doi.org/10.1016/j.cagd.2026.102550) |

### Chronology and development

These five papers form two related tracks of work by Hua Tong (with
Yongjie Jessica Zhang, and Eni Halilaj on the first) at CMU, tracked here
in the order the work was actually done (not filename year):

1. **2024-03 — `HybridOctree_Hex`** (*J. Comput. Sci.*). The earliest work
   and the foundation: an octree-based pipeline for generating adaptive
   all-hex meshes from a surface boundary, with curvature/narrow-region
   detection, a strongly-balanced octree, dual mesh construction, and a
   quality-improvement step (Laplacian smoothing + Jacobian/scaled-Jacobian
   optimization) folded in as post-processing to reach min SJ > 0.5.
   Repo: [CMU-CBML/HybridOctree_Hex](https://github.com/CMU-CBML/HybridOctree_Hex).

2. **2024-10 — `HexOpt`** arXiv preprint, *"Fast and Robust Hexahedral
   Mesh Optimization via Augmented Lagrangian, L-BFGS, and Line Search"*.
   Pulls the quality-improvement half of `HybridOctree_Hex` out into its
   own standalone optimization package, generalized to take *any*
   triangulated surface + hex mesh pair (not just octree-generated ones).
   Innovations over the 2024 J. Comput. Sci. work: a formal constrained
   optimization formulation (objective = rectified scaled Jacobian,
   equality constraints = surface-fitting projection), solved with the
   Augmented Lagrangian method, L-BFGS in place of fixed-learning-rate
   steepest descent, and an Armijo line search for step size. **This is
   the arXiv preprint of the algorithm implemented in this repo.**

3. **2026-01 — `Element-Saving Hexahedral 3-Refinement Templates`**
   arXiv preprint. A separate line of work (mesh *generation*/refinement,
   not optimization): introduces a 3-refinement template set for
   adaptive octree grids under a "moderately-balanced" condition, looser
   than prior 3-refinement requirements, to cut element counts and
   preserve min SJ relative to both prior 3-refinement methods and
   state-of-the-art 2-refinement octree approaches. Addresses the
   "requires strict compatibility conditions that over-refine the grid"
   limitation noted as future work in `HybridOctree_Hex`.

4. **2026-03 — `HexOpt` journal version** (*Computer-Aided Design*,
   published online 2026-03-28). The peer-reviewed, final version of the
   2024-10 arXiv preprint (item 2), for the same
   [CMU-CBML/HexOpt](https://github.com/CMU-CBML/HexOpt) repository this
   fork tracks. Innovations over the 2024 preprint: the objective
   function is reformulated from a plain rectified scaled Jacobian into
   a **Rectified Hybrid Quadratic Jacobian (ReHQJ)**, combining Jacobian
   and scaled-Jacobian metrics as rectified, quadratic measures to fix
   problematic behavior in degenerate regions; adds **geometry-aware
   mapping** (closest-point projection with mean value coordinate
   embedding) for higher-fidelity boundary fitting; and reports
   experimental results across ~100 models surpassing prior
   state-of-the-art, including on curved/sharp-feature geometries.
   **This paper is the main reference for this forked repo.** —
   see the earlier discussion below for why.

5. **2026-04 — `Element-saving hexahedral 3-refinement templates`
   journal version** (*Computer Aided Geometric Design*, published
   online 2026-04-13). The peer-reviewed version of item 3, adding two
   open-sourced variants (one optimized for speed, one for further
   reduced element count) and additional comparison against 2-refinement
   state-of-the-art on Hausdorff ratio.

**For working with this repo's source code specifically**, refer
primarily to paper 4 (`Tong 2026 HexOpt Rectified Hybrid Quadratic
Jacobian.pdf`) — it's the published, final description of the exact
method `meshQuality.cpp`/`optimizer.cpp` implement (AL + L-BFGS +
Armijo line search driving a ReHQJ objective, with Laplacian smoothing).
Paper 2 (the 2024 arXiv preprint) is useful background for how the
method evolved but describes an earlier, superseded objective function.

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
