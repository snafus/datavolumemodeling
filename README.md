# SRCNet Storage Calculator

An interactive, single-file storage estimator for radio astronomy HPC/HTC/cloud nodes in the SKA Regional Centre Network (SRCNet). It sizes the five storage pools a node needs — **T0 fast POSIX**, **T1 bulk Rucio**, **Project space**, **Scratch**, and **User space** — from a small set of parameterised inputs, and checks that they fit within the node's pledged share of the SRCNet fabric.

It is a **capacity planning aid**. All figures are *usable/fillable* capacity. Data protection (replication, erasure coding, RAID) and I/O throughput are deliberately out of scope (see notes in the tool).

## Live version

The calculator is a single self-contained HTML file (no build step, no dependencies, no backend). It is published via GitHub Pages:

- **`index.html`** is the Pages entry point.
- Once Pages is enabled, it is served at `https://<user-or-org>.github.io/<repo>/`.

### Enabling GitHub Pages

1. Push this repository to GitHub.
2. Go to **Settings → Pages**.
3. Under *Build and deployment*, set **Source = Deploy from a branch**.
4. Select the branch (e.g. `main`) and folder `/ (root)`, then **Save**.
5. After ~1 minute the calculator is live at the URL above.

Every push to the branch updates the live version, so the model can be reviewed and calibrated through normal pull requests with full version history.

### Embedding in Confluence

Two options:

- **iframe macro (recommended):** point an iframe macro at the GitHub Pages URL. Height ~1100px. The embed stays in sync with the repository automatically.
- **HTML macro:** paste the contents of the `<body>` tag into a Confluence HTML macro. Note that some Confluence Cloud instances restrict inline JavaScript in the HTML macro; if so, use the iframe approach.

## The model

The node receives an **online allocation** equal to `SRCNet Online Total × node share`. This allocation is partitioned into distinct pools, with T1 absorbing the remainder:

```
Node online allocation = T0 + Project + User + T1 (remainder)
```

| Pool | Formula | Scaling driver |
|------|---------|----------------|
| **Project space** | `active projects × nominal PB/project` | Number of projects (compute-independent) |
| **T0 fast POSIX** | `Project space × T0:Project ratio` | Project space, plus staging/working headroom |
| **User space** | `users × quota × backup factor` | Headcount + backup overhead |
| **T1 bulk Rucio** | `allocation − (T0 + Project + User)` | Remainder of the allocation |
| **Scratch** | `scratch_per_PFLOP × PFLOPS × pipeline multiplier` | Compute (separate filesystem, **not** part of the allocation) |

### Key modelling decisions

- **Project space is the active working area** — a nominal allocation per project driven by retained science data products, not by compute.
- **T0 is the same active data on fast storage**, sized as Project space × a tunable ratio (default 1.2). The ratio expresses staging/working headroom.
- **T1 is the bulk remainder** — whatever capacity is left in the allocation after the sized pools are carved out. A negative remainder signals over-allocation.
- **Scratch is the only compute-driven pool** and is a *separate* fast filesystem, outside the online allocation.
- **Backup overhead applies to user space only**, as an additional usable-capacity factor for snapshots, versioning, and copy-on-write retention.

### Reference baseline

Default ratios derive from the SKA SRC reference point: **10 PFLOPS → 270 PB online storage, ~5 PB scratch, 50,000 cores** (≈ 0.5 PB/PFLOP scratch, 5,000 cores/PFLOP). T2 tape storage is excluded from the current model.

## Tunable parameters

All ratios and thresholds are editable in the **Advanced / calibration** panel inside the tool. To ship a calibrated version, edit the default values in the `CALIB_DEFAULTS` object (and the corresponding `value="…"` attributes) in `index.html`, then commit.

| Parameter | Default | Unit |
|-----------|---------|------|
| Nominal space per project | 0.5 | PB/project |
| T0 : Project ratio | 1.2 | × |
| Scratch per PFLOP | 0.5 | PB/PFLOP |
| Cores per PFLOP | 5000 | cores/PFLOP |
| User quota | 0.5 | TB/user |
| Scratch warn / critical | 100 / 200 | GB/core |
| Allocation-use warning | 90 | % of allocation |

Pipeline types (Continuum, Spectral line, Pulsar/transient) apply a multiplier to **scratch only**. A **Custom** option exposes an editable scratch multiplier.

## What this tool does *not* do

- **I/O throughput.** Scratch, T0 staging, and the T1↔T0 interconnect all have bandwidth (GB/s) requirements that are frequently the binding constraint for radio astronomy pipelines — often more limiting than capacity. Assess these separately.
- **Data protection overheads.** Figures are usable capacity; replication/EC/RAID multipliers are not applied.
- **T2 tape.** Excluded from the current model.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The calculator (GitHub Pages entry point) |
| `storage_calculator_v7.html` | Identical source copy, version-named for reference |
| `README.md` | This document |

## License / attribution

Internal planning tool. Ratios are placeholder estimates pending calibration against measured pipeline data volumes and confirmed SRCNet allocations.
