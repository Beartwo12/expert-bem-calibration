[README.md](https://github.com/user-attachments/files/34949268/README.md)
# Models, code and synthetic simulation data for expert BEM calibration

This package accompanies **Data-driven learning of expert calibration for building
energy simulation models**, by Xilong Liao, Baizhan Li, Haoyu Xie, Tim Broyd,
Chuanyu Tang, Nan Li, Guanyu Li and Chenqiu Du, accepted for publication in
*Building Simulation* (2026; manuscript BUIL-D-26-00739).
Paper DOI: <https://doi.org/10.1007/s12273-026-1495-z>.

It provides two calibration workflows: cooling-only calibration, and full-data
calibration using cooling, heating and total-energy signals. The package includes
source scripts, synthetic simulation outputs, preprocessed features, trained
network weights, parameter metadata, and building and weather input files.
The confidential measured real-building energy dataset is not distributed.

## Package layout

```text
Supplementary models, codes and data/
  README.md
  DATA_README.md
  requirements.txt
  LICENSE
  LICENSE-CODE.txt
  LICENSE-DATA.txt
  THIRD_PARTY_NOTICES.md
  CITATION.cff
  Training_model.idf
  Test_model.idf
  common_tools/
  cooling_only/
    common/
    src/
    data/
      simulation_data/
      alternative_simulation_data/
      origin_data/
      generated_data/
  full_data/
    common/
    src/
    data/
      simulation_data/
      origin_data/
      generated_data/
```

`DATA_README.md` describes data layouts, normalization, parameter ordering and
the roles of the supplied model and feature files. Keep each workflow's directory
structure intact: most input paths are resolved relative to that workflow.

## Software setup

Use Python **3.10 or later** and install the Python packages in a separate
environment. The package is configured around Windows-style EnergyPlus paths.

```bash
python -m venv .venv
```

Activate that environment using the command appropriate to your shell, then run:

```bash
python -m pip install -r requirements.txt
```

The supplied requirements are a dependency list, not a lockfile of the original
experimental environment. Install the PyTorch build appropriate to your CPU or
CUDA environment; see <https://pytorch.org/get-started/locally/>. Training and
calibration scripts select a supported acceleration device when available and
otherwise use the CPU.

The source uses pandas' legacy uppercase `H` frequency alias. The requirements
therefore select pandas 2.x; pandas 3.x is not suitable for the unchanged source.
Optional GUI utilities use Tkinter, which must be available in the Python
installation. The `.xls` parameter tables are reference files; reading those
tables separately may require an additional Excel reader such as `xlrd`.

### EnergyPlus

Install **EnergyPlus 24.1** separately; it is not bundled. The supplied IDF files
declare that version. In each calibration script that you intend to run, change
`EP_EXE` from the supplied machine-specific location:

```python
EP_EXE = Path(r"D:\EnergyPlusV24-1-0\EnergyPlus.exe")
```

to the path of your EnergyPlus executable. The selected IDF template must remain
compatible with the 13 parameter placeholders consumed by that script. The
default virtual-ground-truth scripts use each workflow's
`data/origin_data/initial.idf`; the root-level IDFs are additional model assets.

## Running a workflow

Extract the archive and open a terminal in the top-level extracted directory.
Paths below are relative to that directory. If the extraction path contains
spaces, quote it when changing directories.

### Use the supplied model for virtual-ground-truth calibration

The `.npz` feature archives, trained `.pth` weights and parameter bounds are
already supplied. After installing dependencies and setting `EP_EXE`, select
one of the following entry points:

```bash
python full_data/src/global_optimization_virtual_gt.py
```

```bash
python cooling_only/src/global_optimization_virtual_gt.py
```

These scripts generate synthetic observations through EnergyPlus and run
multi-start calibration. They do not require the withheld measured energy data.
The scripts produce simulation folders, CSV/Excel metrics and convergence plots;
the output root is printed during execution. Check `NUM_GT_CASES`,
`TOTAL_STARTS` and `CONCURRENT_LIMIT` in the selected script before launching a
large experiment. Reduce concurrency if CPU or memory resources are limited.

### Regenerate features and retrain

Full-data workflow, in order:

```bash
python full_data/src/preprocess_massive_csv.py
python full_data/src/train_delta_converged.py
python full_data/src/global_optimization_virtual_gt.py
```

Cooling-only workflow, in order:

```bash
python cooling_only/src/preprocess_massive_csv.py
python cooling_only/src/train_delta_converged.py
python cooling_only/src/global_optimization_virtual_gt.py
```

Preprocessing reads `data/simulation_data/` and writes features to
`data/origin_data/`. Training writes model weights and normalization metadata to
`data/generated_data/`. These commands can replace the supplied feature and
weight files; work on a separate extracted copy if you need to preserve the
distributed artifacts.

The cooling-only alternative dataset is kept separately in
`cooling_only/data/alternative_simulation_data/`. The default commands do not
read it. Keep its simulation index paired with its own energy data rather than
mixing it with the default feature/model files.

### Calibrate against your own measured cooling data

The `cooling_only/src/global_optimization_real_data*.py` scripts are supplied
for this branch. They expect measured cooling data, with a default path of
`cooling_only/data/origin_data/Cooling.csv`. That file is deliberately absent
because the project's measured operating data are confidential. Supply your own
authorized measurements and configure the building area, normalization,
operating conditions, time alignment and model/weather inputs for your case.

### Supporting utilities

`common_tools/` contains scripts for extracting EnergyPlus cooling outputs,
aggregating results, and summarizing CVRMSE/NMBE and screening success rates.
Several utilities open file/folder selection dialogs through Tkinter. The
EnergyPlus executable, its supporting files and Python's standard-library
modules are not installed by `requirements.txt`.

## Data size and reproducibility

The original research payload contains approximately **31.21 GB** of uncompressed
files. Reserve additional disk space for extraction, preprocessing and repeated
EnergyPlus runs. The raw energy matrices are large even though preprocessing
uses chunked reading. See `DATA_README.md` before editing or interpreting them.

Random sample pairing, train/validation splits, initial states and hardware can
affect the numerical results. Record your Python/dependency versions and run
settings when reporting a reproduction. The documentation update preserves the
distributed source, model and data files; it is not an end-to-end rerun of the
research experiments.

## Reuse and attribution

The license is assigned **by file category**, not as a choice between licenses
for the whole archive:

- Original Python source code: **MIT License**, in `LICENSE-CODE.txt`. Retain
  the copyright notice and permission notice in copies or substantial portions.
- Author-created building model files, trained network weights, synthetic
  simulation data, derived features/parameter metadata and package documentation:
  **CC BY 4.0**, subject to `LICENSE` and the complete terms in `LICENSE-DATA.txt`.
  When sharing these materials or adaptations, give appropriate credit, retain
  supplied notices, link to the source and license, and indicate changes.
- Third-party materials, including weather inputs and external software, retain
  their own rights and terms; see `THIRD_PARTY_NOTICES.md`.

A suitable attribution for the author-created non-code materials is:

> Source: Liao, Li, Xie, Broyd, Tang, Li, Li and Du (2026), supplementary models,
> code and synthetic simulation data for "Data-driven learning of expert
> calibration for building energy simulation models", Building Simulation,
> https://doi.org/10.1007/s12273-026-1495-z. Author-created non-code materials
> licensed under CC BY 4.0, https://creativecommons.org/licenses/by/4.0/.
> State whether changes were made, and link to the repository release used when
> that release has a public URL or DOI.

If this resource contributes to a publication, **please cite the associated paper
and the repository release used**. This scholarly citation request does not add
conditions to the MIT License. Citation metadata for the paper are provided in
`CITATION.cff`; the paper DOI must not be presented as a DOI for this archive.

Liao X, Li B, Xie H, Broyd T, Tang C, Li N, Li G, Du C (2026). Data-driven
learning of expert calibration for building energy simulation models.
*Building Simulation*. <https://doi.org/10.1007/s12273-026-1495-z>.

For data-provenance or configuration questions, contact the corresponding author
through the contact information in the associated paper.
