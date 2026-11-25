# Target Trial Emulation on Real-World CRC Data (AACR GENIE BPC)

A portfolio project demonstrating end-to-end real-world evidence (RWE) methodology—target trial specification, multi-table clinical cohort construction, handling of time-related biases and confounding, pipeline validation controls, and sensitivity analyses—using observational oncology data from the **AACR Project GENIE Biopharma Collaborative (BPC) Colorectal Cancer (CRC) v2.0-public** release.

> **Status:** Project initialized. Phase 1 (data map & pre-specified target trial protocol) in progress. No outcome data have been inspected.

## Primary Research Question

Among patients with metastatic colorectal cancer initiating first-line systemic therapy, is adding **bevacizumab** to a fluoropyrimidine/oxaliplatin or irinotecan chemotherapy backbone associated with different **real-world progression-free survival (rwPFS)** compared with chemotherapy alone?

## Planned Workflow

1. **Phase 1 — Protocol & Data Map:** Document all tables and keys (`docs/data_map.md`) and pre-specify the target trial protocol (`docs/protocol.md`) before inspecting any outcome data.
2. **Phase 2 — Cohort Construction:** Reproducible one-command cohort pipeline, CONSORT-style attrition flow diagram, and baseline characteristics (Table 1 with SMDs).
3. **Phase 3 — Primary & Sensitivity Analyses:** Unadjusted Kaplan-Meier curves, propensity score weighting (IPTW / overlap weighting) with balance diagnostics, weighted Cox / RMST models, multiple imputation, and pre-specified sensitivity analyses (E-value, alternative time-zero/regimen rules, institution stratification).
4. **Phase 4 — Pipeline Controls:** Positive controls (e.g., *BRAF* V600E and stage vs. survival) and optional negative control.
5. **Phase 5 — Report & Reproducibility:** Quarto report with explicit limitations and pinned environment (`renv`).

## Data Access & Terms of Use

**No patient-level data are included in this repository.** The underlying dataset (`syn39433168`, CRC v2.0-public) is controlled-access and governed by the [AACR Project GENIE BPC Terms of Use](https://www.aacr.org/professionals/research/aacr-project-genie/bpc/crc-public-release/).

To reproduce the analysis once complete:
1. Create a [Synapse](https://www.synapse.org/) account (username/password).
2. Request access to `syn39433168` and accept the GENIE BPC Terms of Use.
3. Provide your Synapse Personal Access Token locally (excluded via `.gitignore`).

## License

Code in this repository is licensed under the [MIT License](LICENSE). Data access and use remain subject to the AACR Project GENIE BPC Terms of Use.
