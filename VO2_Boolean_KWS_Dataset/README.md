# VO₂–Boolean Temporal Encoding for Keyword Spotting

**Research data and ASIC model artifacts** for the project *Hybrid VO₂–Boolean Temporal Encoding with a Multiplier-Free Readout for Keyword Spotting*.

This repository contains experimental measurements of a capacitively coupled VO₂ oscillator pair, extracted FPGA Boolean-temporal features for a six-class keyword-spotting task, repeated-run data, and the frozen sparse Shift2 readout model used for the SKY130/ASAP7 ASIC implementation comparisons.

> **Release scope:** This is a curated **data and result-artifact repository**, not a complete collection of experiment-control software, audio recordings, ASIC RTL, routed design files, foundry signoff files, or a self-contained training pipeline. See [Data availability and limitations](#data-availability-and-limitations).

## Repository layout

```text
VO2_Boolean_KWS_Dataset/
├── README.md
├── Device_Characterization/
│   ├── BJT_F.csv
│   ├── OSC.CSV
│   └── VO2_VI.xlsx
├── Coupling_Experiments/
│   ├── OSC_Wave/
│   ├── Static_Bias/
│   │   ├── Uncoupled_0pF/
│   │   └── Coupled/                 # Cc10pF, Cc20pF, Cc30pF
│   └── Dynamic_Bias_Sweep/
│       ├── STARTING_POINTS.csv
│       └── Cc10pF/, Cc20pF/, Cc30pF/, Cc47pF/
├── FPGA_Boolean_Features/
│   └── features_all_raw.csv
├── Repeatability/
│   └── 2dot7k/                    # 36 acquisitions, three prompt CSVs each
└── ASIC/
    ├── README.md
    ├── Model/                       # Sparse Shift2 model parameters
    ├── ROM/                         # Serialized 91 × 40-bit ROM
    ├── Verification/                # Model identity and functional audit
    └── Results/                     # Recognition and PPA comparisons
```

The four non-ASIC data directories preserve their original contents and relative paths. The ASIC contents have been reorganized by function; [ASIC/README.md](ASIC/README.md) lists each file.

## Experimental data

### 1. Device characterization

`Device_Characterization/` includes measured VO₂ current–voltage data (`VO2_VI.xlsx`), oscillator waveform data (`OSC.CSV`), and BJT bias/frequency data (`BJT_F.csv`). The different source formats are preserved without numerical conversion.

### 2. Coupling experiments

`Coupling_Experiments/OSC_Wave/` contains oscilloscope CSV traces and a Tektronix setup file.

**Static bias (Task 1):** `Static_Bias/` contains 200 capture NPZ files (50 per coupling condition): uncoupled `0 pF`, and coupled `10`, `20`, and `30 pF`. Each run directory contains an acquisition `metadata.json`. Representative NPZ keys include `event_words`, `sample_rate_hz`, `capture_ms`, `sig_a_packed`, `sig_b_packed`, `a_rising_clk`, `b_rising_clk`, `dac_a_requested_v`, `dac_a_code`, and `dac_a_quantized_v`.

**Dynamic bias sweep (Task 2):** `Dynamic_Bias_Sweep/` contains **231 capture NPZ files**: `10 pF` (54), `20 pF` (47), `30 pF` (59), and `47 pF` (71), plus `STARTING_POINTS.csv`.

**Important voltage convention:** Dynamic-sweep NPZ files store **relative DAC A voltage**, not the absolute DAC setting:

- `delta_va_v`: realized/quantized change in DAC A voltage relative to the first retained capture in the same coupling condition, in volts.
- `delta_va_requested_v`: requested relative change, in volts.
- `metadata_json`: acquisition parameters and the relative-voltage zero-reference definition.
- `STARTING_POINTS.csv`: each sweep's first requested and quantized **absolute** DAC A voltage.

Recover the absolute setting with:

```text
DAC_A_absolute_V = STARTING_POINTS.first_quantized_va_v + capture.delta_va_v
```

For the included sweeps, the initial request is `0.8 V` and the quantized initial value is `0.799755812 V`. Use the matching `cc_pf`/`run_name` row of `STARTING_POINTS.csv`; do not mistake the stored `delta_va_v` for an absolute bias. The static-bias NPZ files, by contrast, include explicit absolute DAC A fields.

The NPZ arrays contain hardware capture data such as packed signal records, event words, and rising-edge indices. Event-word bit meanings and packed-bit order require the original acquisition/FPGA protocol; they are not fully documented by this data release alone.

### 3. FPGA Boolean-temporal features

`FPGA_Boolean_Features/features_all_raw.csv` contains **1,023 source clips**, one row per source clip, with **2,520 numerical feature columns** plus nine metadata columns (**2,529 columns total**).

Feature organization:

- Four audio paths: `full`, `low`, `mid`, `high` (digitally generated RC-like filter paths for the KWS acquisition, not physical RC filters in the KWS signal path).
- Three hardware-acquisition repeats per source clip/path: `r0`, `r1`, `r2`.
- Thirty 50-ms temporal segments per 1.5-second recording.
- Seven features per segment: `and_edge`, `and_high`, `xor_edge`, `xnor_edge`, `xnor_high`, `nor_edge`, `nor_high`.

Thus `4 paths × 3 repeats × 30 segments × 7 features = 2,520` numerical feature columns. Headers follow patterns such as `full__r0__seg00__and_edge`.

The six labels are `hey_android`, `hey_snapdragon`, `hi_galaxy`, `hi_lumina`, `noise`, and `unknown`. `relative_path_original` and `audio_export_path` refer to the source recording organization; **the WAV/audio files themselves are not bundled**.

The finalized ASIC readout selects the **AND+NOR** subset: `4 paths × 30 segments × 4 features = 480` values per selected acquisition repeat (AND/NOR rising-edge and high-duration counts).

### 4. Repeatability

`Repeatability/2dot7k/` preserves 36 timestamped acquisition directories, each with three `index,count` CSV files named for `HELLO`, `HI_SIRI`, and `NI_HAO`.

## ASIC model and results

The fixed, speaker-disjoint ASIC model uses 480 AND+NOR features, a sparse **Shift2** readout, six classification lanes, and a **91-row × 40-bit** serialized ROM (455 bytes). The included training record identifies the exported exemplar as **speaker Split 1 (`s000`), acquisition repeat `r0`, and regularization `C = 3.5 × 10⁻⁵`**.

For that selected model the archived records report **149/153 = 97.39% validation accuracy** and **138/152 = 90.79% held-out test accuracy**. These are **one selected model's results**, not the mean accuracy over all speaker partitions. The per-repeat results table provides 30 runs (10 speaker partitions × 3 repeats). The speaker split is 35 training / 8 validation / 7 test speakers.

The archived SKY130 and ASAP7 comparisons use a nominal 100-MHz backend with 221 compute cycles (2.21 µs) and a 200-kHz always-on control clock. The reported power, area, and timing values are **post-route estimates** under differing technology/library assumptions, **not measured silicon** and **not complete foundry signoff**. Interpret the estimates alongside `ASIC/Verification/ASAP7_UNRESOLVED_LIMITATIONS.md` and the qualification fields of `ASIC/Results/SKY130_VS_ASAP7_FINAL_COMPARISON.csv`.

See **[ASIC/README.md](ASIC/README.md)** for file-by-file descriptions, ROM layout, and audit caveats.

## Minimal data access examples

Python 3 with NumPy can open the measurement NPZ files:

```python
from pathlib import Path
import csv
import json
import numpy as np

root = Path("VO2_Boolean_KWS_Dataset")
run_dir = root / "Coupling_Experiments" / "Dynamic_Bias_Sweep"
npz_path = next((run_dir / "Cc10pF").rglob("capture_0000.npz"))

with np.load(npz_path, allow_pickle=False) as capture:
    delta_v = float(capture["delta_va_v"])
    details = json.loads(str(capture["metadata_json"].item()))

with (run_dir / "STARTING_POINTS.csv").open(newline="") as handle:
    starts = list(csv.DictReader(handle))
start = next(
    row for row in starts
    if float(row["cc_pf"]) == float(details["cc_pf"])
    and row["run_name"] == details["sweep_run"]
)
absolute_v = float(start["first_quantized_va_v"]) + delta_v
print("Relative DAC A (V):", delta_v)
print("Absolute DAC A (V):", absolute_v)
```

For the feature matrix:

```python
import pandas as pd
frame = pd.read_csv("VO2_Boolean_KWS_Dataset/FPGA_Boolean_Features/features_all_raw.csv")
feature_cols = [c for c in frame if "__seg" in c]
print(frame.shape)       # (1023, 2529)
print(len(feature_cols)) # 2520
```

The pandas example is optional; `csv.DictReader` in the Python standard library also works.

## Data availability and limitations

- Included: VO₂/device measurements, FPGA event captures, processed Boolean-feature table, repeated acquisitions, frozen ASIC model coefficients/scales/biases/ROM, identity/functional verification summaries, and published-analysis-facing comparison tables.
- Not included: source audio/WAV files, complete training scripts and training-model NPZ artifacts, acquisition firmware/FPGA gateware, ASIC RTL/testbenches, synthesis/place-and-route scripts, GDS/DEF/SPEF/SDF, and full foundry-qualified signoff collateral.
- The model-identity JSON and audit notes retain original local filesystem paths and references to archival artifacts **outside this repository**. They are provenance information, not portable paths or included dependencies.
- Hashes in `ASIC/Verification/SHA256_AUDIT.txt` identify the source model, derived handoff, and included ROM. Source-model and handoff NPZ files are **not** included; the included ROM can be independently hashed.
- No license file is supplied in this export. Specify a license and citation details before public release if redistribution or reuse is intended.

## Integrity of this organization

All **563 original archive files** have been retained **byte-for-byte**. Only the 12 ASIC file locations were reorganized; the classification CSV originally named `S5_AND_NOR_per_repeat_30_fits_publication.csv` is now `ASIC/Results/AND_NOR_30_FITS.csv` (same contents). `README.md` and `ASIC/README.md` are newly written. No non-ASIC data path or measurement value was changed.
