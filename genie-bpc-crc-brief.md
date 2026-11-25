# Project brief: Target trial emulation on real-world CRC data (AACR GENIE BPC)

This brief is written for an AI coding assistant working with me (a scientist moving into clinical data science). The repo is a public portfolio project. Its purpose is to show real-world-evidence (RWE) methodology on messy observational oncology data, not to produce a clinical finding.

## 0. Working agreement

- **Before building anything, summarize your plan back to me and wait for my go.** After that, work phase by phase. At the end of each phase, stop and show what was produced.
- **Never guess variable names, table names or code lists.** Read the data dictionary / Analytic Data Guide first and quote the exact names you rely on. If something is unclear, ask me.
- **Do not look at outcome data (survival or progression) until the protocol (Phase 1) is written and committed.** The git commit timestamp is the evidence that the analysis plan came first.
- **Do not claim causality.** Write "associated with" and describe residual confounding honestly.
- **The data are not redistributable.** Never commit data files, Synapse tokens or credentials. Commit code and aggregate results only.

## 1. Goal

Emulate a hypothetical randomized trial using observational data, end to end, as an RWE team would:

1. Write a target trial protocol.
2. Build the analytic cohort from multi-table clinical data.
3. Estimate the effect with explicit handling of confounding and time-related biases.
4. Check the pipeline with known associations.
5. Stress-test the result with sensitivity analyses.
6. Report it with limitations stated up front.

## 2. Data

**AACR Project GENIE Biopharma Collaborative (BPC), colorectal cancer cohort, release CRC v2.0-public.**

- 1,485 CRC patients from three institutions (MSKCC, DFCI, VICC).
- Clinical-grade NGS from the GENIE registry (sequenced 2014 to 2018), combined with EHR-curated treatment and outcome data using the PRISSMM framework.
- Real-world endpoints are provided, including rwPFS, rwTTNT, rwTTD and rwTTR. Verify the exact variable names in the guide.
- About 48% of patients are stage IV at diagnosis.
- Synapse ID of the release: `syn39433168`.

**Access.** Create a Synapse account with a username and password (do not log in via Google), open the release page, click "Request Access" and accept the terms of use. Download via the `synapseclient` Python package or the R package `genieBPC` (`pull_data_synapse()`).

**Links**
- Release page: https://www.aacr.org/professionals/research/aacr-project-genie/bpc/crc-public-release/
- Analytic Data Guide (the key document): https://www.aacr.org/wp-content/uploads/2022/11/GENIE-BPC-CRC-v2.0-public-Analytic-Data-Guide-v2.pdf
- `genieBPC` R package: https://github.com/GENIE-BPC/genieBPC
- Data processing pipeline paper: Lavery et al., "A data processing pipeline for the AACR project GENIE biopharma collaborative"

**Structure to expect.** Several datasets with different units of analysis (patient level, cancer level for index and non-index cancers, drug regimen level, imaging, pathology, NGS panel tests, and others). Part of the work is joining these correctly. Read the guide to learn which unit each table uses.

## 3. Primary research question (default, can be changed)

Among patients with metastatic CRC who start first-line systemic therapy, is adding **bevacizumab** to a fluoropyrimidine/oxaliplatin or irinotecan chemotherapy backbone associated with different **real-world progression-free survival (rwPFS)** than chemotherapy alone?

Secondary (only after the primary analysis is complete): does the association differ by molecular subgroup (RAS, BRAF V600E, MSI status)? Panels differ in gene coverage, so mutation status may be missing or unassessable for part of the cohort. Handle that explicitly and do not treat "not tested" as "wild type".

*Alternative question if this one is underpowered or the data do not support it:* a prognostic analysis of molecular subtype (RAS, BRAF, MSI) versus rwPFS and overall survival in first-line metastatic patients. Decide with me before switching.

## 4. Phases, deliverables and acceptance checks

### Phase 1: Protocol and data exploration (about 10 h)
- Get access, download the data, load every table, and write a **data map** (`docs/data_map.md`): table, unit of analysis, key columns, row counts.
- Write `docs/protocol.md` as a target trial specification with these elements:
  - Eligibility criteria
  - Treatment strategies
  - Treatment assignment
  - **Time zero**
  - Follow-up and censoring
  - Outcome definition (rwPFS) and how death is handled
  - Causal contrast
  - Confounders (with a short DAG or justification)
  - Statistical analysis plan, including the sensitivity analyses in Phase 3
- **Acceptance:** protocol committed before any outcome analysis. Every variable in the protocol exists in the data and is cited from the guide.

**Biases the protocol must address explicitly**
- **Left truncation / immortal time.** Patients enter GENIE only after a tumor sample is sequenced. If sequencing happened after first-line therapy started, patients had to survive to be sequenced. Either restrict to patients sequenced before first-line start, or use delayed entry (risk set starts at NGS date). Describe which you chose and why.
- **Time zero.** Use start of first-line therapy for both arms, so the arms are aligned.
- **Confounding by indication.** Examples: performance status is probably unavailable, comorbidity is probably unavailable, and institution and calendar year are confounders.
- **Treatment misclassification.** Define "first line", "regimen" and "bevacizumab added" from the drug tables with a documented rule. Test the rule on edge cases (regimen switches within a short window, maintenance therapy).

### Phase 2: Cohort construction (about 6 h)
- Reproducible script or notebook that builds the analytic cohort from raw tables.
- A **flow diagram** (CONSORT-style) with counts at every exclusion step, saved as an image and as a CSV.
- A baseline characteristics table by arm (Table 1) with standardized mean differences.
- **Acceptance:** the cohort can be rebuilt from scratch with one command. Counts match the flow diagram.

### Phase 3: Analysis (about 12 h)
- Unadjusted Kaplan-Meier curves with risk tables and median rwPFS with confidence intervals.
- Propensity score model, with IPTW (or overlap weighting). Report covariate balance before and after weighting (love plot). Check positivity and weight distribution.
- Weighted Cox model with robust variance. Report the hazard ratio with 95% CI. Check proportional hazards and, if violated, report restricted mean survival time differences as well.
- Missing data: document missingness per covariate, use multiple imputation, and compare to a complete-case analysis.
- **Sensitivity analyses:** E-value for unmeasured confounding, alternative time-zero or entry definitions, alternative regimen definitions, institution-stratified analysis, and exclusion of the earliest calendar years.
- **Acceptance:** every analysis follows the protocol. Anything not in the protocol is labeled "post hoc".

### Phase 4: Pipeline checks (about 4 h)
- **Positive controls:** reproduce known prognostic associations in this cohort, for example BRAF V600E and stage versus survival, in the expected direction. If the pipeline cannot recover them, find out why before trusting the primary result.
- Optional **negative control**: an exposure or outcome pairing where no effect is expected, to detect residual bias.
- **Acceptance:** control results are reported whether or not they look good.

### Phase 5: Write-up and repo (about 8 h)
- Quarto (or equivalent) report: question, protocol summary, flow diagram, Table 1, results, sensitivity analyses, controls, **limitations**, conclusion.
- README with a project summary, how to reproduce, data access instructions (not the data), and a short methods overview.
- Pinned dependencies (`renv` for R, or `uv` / `requirements.txt` for Python), a `.gitignore` that excludes all data and tokens, a license for the code, and a note on the BPC terms of use.
- **Acceptance:** a stranger can clone the repo, add their own Synapse token, run one command and regenerate the report.

## 5. Tech stack

- **Default: R** for the analysis (`genieBPC`, `survival`, `survminer`, `WeightIt`, `cobalt`, `mice`, `tidyverse`, `renv`), because RWD job ads often list R or SAS. Python (`synapseclient`, `lifelines`, `pandas`) is fine for data retrieval or if I decide to switch. Ask me before changing the language.
- Reporting with Quarto. Version control with git, with small and meaningful commits.

## 6. Limitations to state in the final report

- The public release is relatively small (1,485 patients), so subgroup analyses are underpowered.
- Sequencing took place between 2014 and 2018, so treatment patterns are not current.
- Curated real-world endpoints (rwPFS) are less precise than trial endpoints, because progression assessment depends on routine imaging and clinician documentation.
- Selection into GENIE (patients who were sequenced at academic centers) limits generalizability.
- No randomization, so residual confounding remains even after weighting.

## 7. Definition of done

A public repo containing a committed protocol that predates the outcome analysis, a reproducible cohort and analysis pipeline, a report with pipeline controls and sensitivity analyses, and an honest limitations section.

## 8. Stretch goals (only after everything above is done)

- Replicate the pipeline on the NSCLC cohort (also public) to show the workflow generalizes.
- Compare the results with published trial estimates for similar populations and discuss differences.
- Add a short "what would I do with more data" section.
