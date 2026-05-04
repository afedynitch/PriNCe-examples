# PriNCe-examples

This repository contains examples for the use of the PriNCe propagation code.

Requires the base [PriNCe code](https://github.com/joheinze/PriNCe) and the [PriNCe analysis tools](https://github.com/joheinze/PriNCe-analysis-tools). The latter is optional, since it is only used for plotting measured spectra.

## FLUKA-wiring update (2026-05)

The notebooks in this repo have been ported to the
[`fluka-photo-nuclear-wiring`](https://github.com/joheinze/PriNCe/tree/fluka-photo-nuclear-wiring)
branch of PriNCe and the FLUKA-derived photo-nuclear database produced by
[`prince-fluka-utils`](https://github.com/afedynitch/prince-fluka-utils).
Highlights of the API change consumed here:

- Photo-nuclear cross sections now come from a single class,
  `cross_sections.FlukaPhotoNuclear()`, replacing the
  `CompositeCrossSection([(0., TabulatedCrossSection,…), (0.14, SophiaSuperposition,())])`
  recipe and the standalone `TabulatedCrossSection('PEANUT_IAS' | 'CRP2_TALYS' | 'PSB')`
  / `SophiaSuperposition()` / `EmpiricalModel()` constructors (all dropped on
  the wiring branch).
- The solver is `UHECRPropagationSolverETD2` (vendored Cox–Matthews ETD2);
  `UHECRPropagationSolverBDF` and `UHECRPropagationSolverEULER` are gone.
- Species IDs are PDG, not the legacy NCo encoding. Source-class params now
  use `2212` / `1000020040` / `1000070140` / `1000140280` / `1000260560`
  (rather than `101` / `402` / `1407` / `2814` / `5626`); plotting filters
  read `pdgid2sref` rather than `ncoid2sref`.

### Pointing the notebooks at the FLUKA database

Each notebook starts with a small prelude that selects the FLUKA-derived
HDF5 database — `prince_db_v1.h5`, built by `prince-fluka-utils`:

```python
import prince_cr.config
prince_cr.config.fluka_db_path = '/path/to/dir/containing/prince_db_v1.h5'
prince_cr.config.fluka_db_fname = 'prince_db_v1.h5'
```

The committed default points at the SATORI v1 production build at
`/ceph/sharedfs/work/SATORI/anatoli/devel/UH-UHECR-Fluka-Prince/runs/2026-05-04_pfu-v1-prod`.
Override these two settings for any other machine, or copy the HDF5 file
into PriNCe's `data/` directory and unset `fluka_db_path` (which defaults
to `data_dir`).

The legacy `prince_db_05.h5` file is still required for EBL splines; PriNCe
will auto-download it on first import.

## How to use

1. Install the FLUKA-wiring branch of PriNCe (e.g.
   `pip install -e /path/to/PriNCe` after `git checkout fluka-photo-nuclear-wiring`).
2. Build the FLUKA database via `prince-fluka-utils` (or copy a prebuilt
   `prince_db_v1.h5`), and point `prince_cr.config.fluka_db_path` at it
   (see prelude above).
3. Install `jupyter notebook` or `jupyter lab`.
4. Clone this repository.
5. Launch `jupyter lab/notebook` and visit the notebooks in the folders.

Some notes on performance:

- Kernel construction at `max_mass=56` runs in ~5 s on a laptop with the
  toeplitz path landed in PriNCe `worktree-kernel-accel-explore` (now on
  `master` of the wiring branch). Larger `max_mass` (up to 245 in the v1
  database) scales accordingly.
- The integration step scales across multiple cores and benefits from GPU
  support if available; with ETD2 the per-step cost is dominated by 4
  sparse-matrix-vector multiplies.

## Authors

- *Anatoli Fedynitch*
- *Jonas Heinze*

## Copyright and license

Code released under [the BSD 3-clause license (see LICENSE)](LICENSE).
