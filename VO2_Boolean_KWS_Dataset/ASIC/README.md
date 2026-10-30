# ASIC model, validation, and post-route results

This folder organizes the **12 existing ASIC-related artifacts** into functional categories. It contains a **frozen model export and reported verification/results**, rather than the complete RTL/physical-design project.

## File index

| Location | File | Meaning |
| --- | --- | --- |
| `Model/` | `FINAL_ASIC_SHIFT2_PARAMETERS.csv` | 480-feature mapping, active-row/ROM indices, group and quantized-coefficient metadata. |
| `Model/` | `SIX_CLASS_COEFFICIENTS_6x480.csv` | Six-class sparse Shift2 coefficients in CSV form. |
| `Model/` | `GROUP_SCALES.csv` | The 16 path/Boolean-group scales, including float32 and Q60 representations. |
| `Model/` | `CLASS_BIASES.csv` | Six class biases in float32 and Q60 representations. |
| `ROM/` | `ROM_PAYLOAD_91x40.bin` | 455-byte binary: 91 consecutive unsigned 40-bit ROM words, big-endian within each 5-byte word. |
| `ROM/` | `ROM_PAYLOAD_91x40.hex` | One 10-hex-digit ROM word per line, in matching address order. |
| `Verification/` | `MODEL_MATCH_EVIDENCE.json` | Frozen training-model identity, split/repeat/C selection, model hashes and test/validation metrics. |
| `Verification/` | `SHA256_AUDIT.txt` | Recorded hashes for source model NPZ, handoff NPZ, and included ROM. |
| `Verification/` | `ASAP7_FUNCTIONAL_VERIFICATION.md` | Recorded ASIC equivalence, clock protocol, and regression coverage. |
| `Verification/` | `ASAP7_UNRESOLVED_LIMITATIONS.md` | Explicit limits for ASAP7/SKY130 timing, power modeling, DRC/LVS/SDF, and signoff. |
| `Results/` | `AND_NOR_30_FITS.csv` | Ten speaker splits × three acquisition repeats: accuracy and Shift2 sparsity results. |
| `Results/` | `SKY130_VS_ASAP7_FINAL_COMPARISON.csv` | Comparison of the two post-route ASIC implementation estimates with source notes. |

The file `Results/AND_NOR_30_FITS.csv` was formerly named `S5_AND_NOR_per_repeat_30_fits_publication.csv`. **Only its name and location changed; its bytes were not altered.** There are no `provenance/` or `supplementary_data/` directories in this release.

## Frozen model snapshot

- Six classes: `hey_android`, `hey_snapdragon`, `hi_galaxy`, `hi_lumina`, `noise`, `unknown`.
- AND+NOR, four audio paths, 30 temporal segments; 480 candidate input features.
- 91 active ROM rows, 389 inactive rows; 91 × 40-bit ROM = 455 bytes; six class coefficient lanes.
- Speaker Split 1 (`shuffle_id_zero_based = 0`/`s000`), four-path acquisition `r0`, `C = 3.5e-05`.
- Recorded validation: 149 / 153 = 97.39%; recorded test: 138 / 152 = 90.79%.
- Frozen design backend: 100 MHz, 221 cycles (2.21 µs compute), with 200-kHz always-on control.

These selected-model percentages should not be confused with mean performance over the 30 model fits or with exact **functional equivalence** (matching software-derived scores for all tested samples).

## ROM integrity check

The archived `SHA256_AUDIT.txt` reports the ROM hash:

```text
bea1c20eae7b0ac2abb32271b2ac103b9880539ef08c56612973e47d9b72c3ea
```

For example:

```bash
sha256sum ASIC/ROM/ROM_PAYLOAD_91x40.bin
```

`SHA256_AUDIT.txt` and the existing verification Markdown are preserved as source evidence. They may use **legacy names and external archive paths** such as `provenance/`, `FINAL_ASIC_MODEL_PROVENANCE/`, and workstation paths: those are original references, not additional files included in this GitHub dataset. Use the current paths in the table above to locate included files.

## Verification qualification

The archived regression reports record agreement on 152/152 predictions, 912/912 score comparisons, and 13,832 feature/weight alignments for the frozen ASIC model. This is **model-to-implementation functional agreement** on checked cases, not classification accuracy of 100%, and does not establish complete sequential formal equivalence.

The PPA values are post-route estimates, **not silicon measurements**. No full timing-annotated SDF simulation or complete foundry signoff is claimed; inspect `Verification/ASAP7_UNRESOLVED_LIMITATIONS.md` and the comparison CSV for details. Source RTL, gate netlists, GDS, and the original training/handoff NPZ files are not present in this repository.
