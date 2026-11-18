# Tumor Growth Analysis

A free, browser-based tool for analyzing caliper tumor measurements: volume calculation, group growth curves, and statistics for whole curves and individual days.

**Use it online:** https://g864245133-ops.github.io/tumor-growth/

**Author:** Xiaoyue Gao, University of Wisconsin-Madison ([ORCID 0000-0002-5414-403X](https://orcid.org/0000-0002-5414-403X))

## Input (long format, one row per measurement)

| Sample | Group | Width | Length | Date | Day_injection | Day_treatment |
|---|---|---|---|---|---|---|
| M1 | Vehicle | 4.10 | 5.62 | 2026-03-09 | 7 | 0 |
| M1 | Vehicle | 4.58 | 6.20 | 2026-03-11 | 9 | 2 |

After loading, groups and samples can be **renamed and reordered** in the "Rename / reorder groups and samples" panel (the original file is not changed; giving two groups the same name merges them).

Sample, Group, Width and Length are required, plus at least one of Date / Day from injection / Day from treatment. Paste from Excel or upload `.xlsx / .csv / .txt`. Click **Load example data** on the page to see a worked example.

### Built-in data converter (lab-notebook layout)

If you record measurements in blocks, with dates across the columns and L / W under each date, click **"My data has dates across columns → convert it"**:

```
OE        10.14          10.16          10.19
          L      W       L      W       L      W
white     5.1    4.0     6.0    4.4     9.3    6.5
red       ...
(blank row)
Control   10.14          10.16          10.19
          L      W       L      W       L      W
white     ...
```

- Upload the `.xlsx` (any sheet) or paste the copied cells. Several group blocks on one sheet are detected automatically.
- Dates such as `10.14` are read as Month.Day (Day.Month can be selected) in the chosen year. When Excel turned `10.10` into `10.1`, the reading that keeps the dates in order is chosen and flagged for checking. A year change (Dec → Jan) is handled.
- Enter the tumor injection and treatment start dates (for all groups or per group) to add `Day_injection` and `Day_treatment` columns.
- Empty cells are skipped. Half-filled pairs (L without W, or W without L) are reported.
- Click **Use this data in the analysis** to load it, or copy the converted table.

## Tumor volume

- Default: **V = L × W² / 2** (modified ellipsoid formula). Alternative: **V = π/6 × L × W²**.
- L = longest diameter, W = perpendicular shorter diameter. If W > L in a row, the two are swapped automatically and flagged.

## Statistical methods

### 1. Whole-curve comparison (growth trend): permutation test

**Primary test.** This is the main answer to "do the groups grow differently?"

- Method: permutation test of growth curves (Elso et al., 2004, *Genes & Immunity*; Baldwin et al., 2007; equivalent to `statmod::compareGrowthCurves` in R).
- For each pair of groups, a Welch t-statistic is computed at every time point and averaged over time ("mean t").
- Group labels are randomly re-assigned among animals (10,000 permutations by default, or every possible permutation when there are fewer). Each animal's whole curve stays together, so the repeated-measures structure is preserved.
- p = proportion of permuted statistics at least as extreme as the observed one.
- With **3 or more groups**, an overall test uses the time-averaged one-way ANOVA F-statistic, permuted the same way.
- Pairwise p-values are **Holm-adjusted**.
- Time points where a group has fewer than 2 animals are skipped, so animals that leave the study early are handled.
- Why: no assumption of normality or of a particular curve shape, valid for the small group sizes typical of mouse studies, and robust to dropouts.

### 1b. Linear mixed-effects model

- ln(volume) ~ group + time + group × time, with a random intercept and slope for each animal (maximum likelihood), the approach used by TumGrowth (Enot et al., 2018).
- Likelihood-ratio tests: **group × time** (do growth rates differ?) and **group overall** (any difference in level or rate). Pairwise tests are Holm-adjusted.
- Uses every measurement; animals with missing days or early removal keep their data. Model-based growth rate per group (± SE) and doubling time are reported.

### 1c. Two-way repeated-measures ANOVA

- Mixed-design (split-plot) ANOVA, group × time, with Greenhouse-Geisser correction (as in GraphPad Prism).
- Uses days up to the last common day, and only animals with complete data there.

### 1d. Time to endpoint: Kaplan-Meier and log-rank

- Event = first day the tumor reaches a threshold: a fixed volume (mm³) or a multiple of its own baseline (e.g. 4× = quadrupling time, which does not depend on baseline). Estimated by log-linear interpolation. Animals that never reach it are censored.
- Log-rank test (overall and pairwise, Holm-adjusted), median time, and a Kaplan-Meier plot.

### 2. Per-animal summary measures (supporting analysis)

Each animal's curve is reduced to one number (summary-measures approach, Matthews et al., 1990, *BMJ*), so each animal contributes exactly one value:

- **Specific growth rate**: slope of ln(V+1) against time (log-linear regression) within the analysis window. It does not depend on starting volume. **Doubling time** = ln2 / growth rate.
- **Area under the curve (AUC)**: trapezoidal rule from the window start to the **last day on which every animal is still measured**, so early removals do not bias it.
- **Tumor growth inhibition** at the last common day: T/C % = mean V(treated) / mean V(control) × 100, and TGI % = (1 − ΔT/ΔC) × 100.
- Compared between groups with Welch's t-test (2 groups) or Welch's ANOVA followed by pairwise Welch t-tests (3+ groups), Holm-adjusted. Non-parametric option: Mann-Whitney U / Kruskal-Wallis.

### 3. Comparison on each day

| Setting | 2 groups | 3+ groups |
|---|---|---|
| Parametric (default) | Welch's t-test | Welch's one-way ANOVA, then pairwise Welch t-tests |
| Non-parametric | Mann-Whitney U (exact when there are no ties) | Kruskal-Wallis, then pairwise Mann-Whitney U |
| ANCOVA (baseline-adjusted) | ln(V+1) adjusted for ln(V₀+1) | Overall ANCOVA F-test, then pairwise ANCOVA |

- Tumor volumes are right-skewed and their variance grows with the mean, so volumes are **log-transformed** before testing by default: ln(V+1), or ln of relative volume.
- Multiple comparisons: **Holm correction** within each day (across pairs), across all days and pairs, or none.
- Comparisons can be all pairs of groups or each group vs. a chosen control.
- Testing each day separately increases the chance of false positives. Use the whole-curve test (section 1) as the primary result and the per-day tests to describe *when* the groups separate.

## Unequal start dates and baseline volumes

### Different start dates

- Animals are aligned by **day from treatment start** (or day from injection), not by calendar date. Each animal's day 0 is its own start, so cohorts enrolled on different dates are directly comparable.
- If some rows have a Date but no day number, the day is filled in from that animal's other rows (date − day = that animal's start date).
- If only dates are given, days can be counted from the earliest date in the study (shared day 0) or from each animal's first measurement.

### Animals measured on different days

- This is detected automatically: if on an average day fewer than 75% of the animals were measured, the tool switches to **interpolation onto a common day grid**.
- Interpolation is linear on the log scale (exponential growth between measurements) and only within each animal's observed range. It never extrapolates.

### Different baseline volumes

- The **baseline check** compares starting volumes (V₀) between groups (test on ln(V₀+1)) and flags an imbalance if p < 0.05 or the group means differ by more than 25%.
- **Normalization options** (Y variable): relative volume V/V₀, % change, increase V − V₀, log2 fold change, or % of the control mean on the same day (T/C %).
- **Choice of baseline V₀**: the measurement at the window start, each animal's first measurement (useful when start dates differ), or the measurement closest to a chosen day.
- Recommended fixes:
  1. **ANCOVA**: adjusts each day's comparison for baseline volume. This is the most efficient adjustment when baselines differ (Vickers & Altman, 2001, *BMJ*).
  2. **Relative tumor volume (V / V₀)** or **% change from baseline**: normalizes each animal to its own starting size.
  3. **Growth rate**: the slope on the log scale is unaffected by starting size.
- V₀ = last measurement on or before the start of the analysis window (or the first measurement inside the window).
- Best practice remains randomizing animals to groups once tumors reach a target size.

## Output

- Growth curves (mean ± SEM/SD, optional individual animals, per-day significance stars, log Y axis, color palettes), plus a per-animal summary plot. Charts can be copied as images.
- Result tables that can be copied into Excel / Prism.
- An auto-generated methods paragraph that updates with your settings.

## References

- Elso CM, et al. (2004). Leishmaniasis host response loci (lmr1–3) modify disease severity through a Th1/Th2-independent pathway. *Genes & Immunity* 5:93–100.
- Baldwin T, et al. (2007). Wound healing response is a major contributor to the severity of cutaneous leishmaniasis in the ear model of infection. *Parasite Immunology* 29:501–513.
- Matthews JNS, et al. (1990). Analysis of serial measurements in medical research. *BMJ* 300:230–235.
- Vickers AJ, Altman DG (2001). Analysing controlled trials with baseline and follow up measurements. *BMJ* 323:1123–1124.
- Enot DP, et al. (2018). TumGrowth: an open-access web tool for the statistical analysis of tumor growth curves. *OncoImmunology* 7:e1462431.
- Holm S (1979). A simple sequentially rejective multiple test procedure. *Scandinavian Journal of Statistics* 6:65–70.

## How to cite

If this tool helps your work, please cite it (GitHub's **"Cite this repository"** button, top right, gives APA and BibTeX formats):

> Gao X. *Tumor Growth Analysis: a browser-based tool for tumor volume and growth-curve statistics* (v1.0.0). 2026. https://github.com/g864245133-ops/tumor-growth

## License

Released under the [MIT License](LICENSE): free to use, modify and share, including commercially, as long as the copyright notice is kept.

### Third-party libraries

Loaded from cdnjs at run time (not bundled in this repository):

- [Chart.js](https://www.chartjs.org/) 4.4.1, MIT License
- [SheetJS Community Edition](https://sheetjs.com/) (xlsx) 0.18.5, Apache License 2.0

## Disclaimer

This tool is provided for research use only, without warranty of any kind. Check that the chosen statistical methods suit your experimental design, and confirm important results with a statistician or established software where appropriate.

## Privacy

All calculations run locally in your browser. Your data is never uploaded.
