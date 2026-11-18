# Tumor Growth Analysis

A free, browser-based tool for analyzing caliper tumor measurements: volume calculation, group growth curves, and statistics for whole curves and individual days.

**Use it online:** https://g864245133-ops.github.io/tumor-growth/

## Input (long format, one row per measurement)

| Sample | Group | Width | Length | Date | Day_injection | Day_treatment |
|---|---|---|---|---|---|---|
| M1 | Vehicle | 4.10 | 5.62 | 2026-03-09 | 7 | 0 |
| M1 | Vehicle | 4.58 | 6.20 | 2026-03-11 | 9 | 2 |

Sample, Group, Width and Length are required, plus at least one of Date / Day from injection / Day from treatment. Missing day numbers are filled in from the Date when possible.

## What it does

- **Volume:** V = L × W² / 2 (or π/6 × L × W²); W and L are swapped automatically if W > L
- **Alignment:** by day from treatment start, day from injection, or days since first measurement, so cohorts started on different dates are comparable; optional log-linear interpolation onto a common day grid when animals were measured on different days
- **Baseline check:** compares starting volumes between groups and suggests a fix (relative volume, ANCOVA, growth rate)
- **Whole-curve test:** permutation test of growth curves (Elso et al. 2004, as in `statmod::compareGrowthCurves`), with Holm correction and an overall test for 3 or more groups
- **Per-animal summaries:** specific growth rate (slope of ln volume), doubling time, and area under the curve
- **Per-day tests:** Welch t-test / Welch ANOVA, Mann-Whitney / Kruskal-Wallis, or ANCOVA adjusted for baseline, with Holm correction
- Growth curve plots (mean ± SEM/SD, individual animals, significance stars), auto-generated methods text, and tables you can copy into Excel / Prism

## Privacy

All calculations run locally in your browser. Your data is never uploaded.
