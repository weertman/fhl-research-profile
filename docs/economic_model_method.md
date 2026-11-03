# FHL economic impact model: method

> Published with the report *A century of research at Friday Harbor Laboratories*. File paths such as `model/...` and `people_and_local/...` are relative to the economic evidence folder of the census pipeline (`fhl_pubs/data/interim/economics/`), which is not in this repository.

This document describes a modeled economic impact study of Friday Harbor Laboratories (FHL) for two regions, Washington State and San Juan County. It follows the structure of UW's own impact reports (Parker Strategy Group, FY2023; Engineering, FY2024):

- spending sources: operations, research, students and visitors, with construction reported separately;
- effects: direct, indirect and induced;
- measures: output, jobs "supported and sustained", labor income, value added, and state and local taxes;
- two bases: the **contribution** of all FHL activity and the **new-money impact** of spending financed from outside the region.

The model uses public sources only. FHL supplied no budget, payroll, user or purchasing data, and no IMPLAN license was used. The inter-industry structure comes from the Washington Office of Financial Management (OFM) 2012 input-output table. The county model regionalizes that table with BLS QCEW data. Induced (household-spending) effects are calibrated, per dollar of labor income, to published IMPLAN results for UW and for the WSF Friday Harbor route.

The model's inputs, results and one-at-a-time sensitivity are published as `data/economics/model_inputs.csv` (every input with its low, central and high value, Monte Carlo support, source and basis), `model_results.csv` (every result by region, basis, spending source, measure and effect, with the Monte Carlo percentiles and the extreme bounds) and `model_sensitivity.csv`. The model makes no network calls. It reads the files written by four input modules (multipliers, operations, users, and research and construction) and saved public tables (Census margins, BEA county tables, the Dean Runyan 2017p report and the PeaceHealth fact sheet). Every output of the model is regenerated on each run, and every number quoted below comes from those outputs.

## 1. Target year and dollars

- **Target:** a typical FY2024 year.
  - Operations: a typical FY2023–FY2025 year.
  - Student and visitor spending: calendar 2024.
  - Research: the average of UW FY2022–FY2024, the three-year window closest to FY2024.
- **Construction:** reported separately, as a one-time 2015–2024 total (job-years) and as its annual average.
- **Dollars:** all results are in **2024 dollars**. Each input module converted nominal amounts with BLS CPI-U annual averages (2024 = 313.689). Where 2025–2026 values were deflated, the modules used the BLS 2025 annual average (321.943) and the January–September 2026 mean (332.243), because the 2026 annual figure is not published. October 2025 CPI-U was not published; the research module interpolated it for fiscal-year averages only.
- **Jobs per dollar** from the 2012 table are converted to 2024 dollars by a blend of QCEW pay growth and CPI-U (section 2.3).

## 2. Regional models

### 2.1 Washington State

The model is the OFM 2012 Washington Input-Output Model (52 sectors). OFM lists only the 2002, 2007 and 2012 models, so 2012 is the latest.

- A = Z / X.
- Type I inverse: L1 = (I − A)⁻¹.
- Type II inverse: L2 closes households as OFM does. The household row is h = labor income per $ of output. The household column is c = PCE purchases from WA industries ÷ total WA labor income ($229,780.5M).

`model.py` rebuilds the matrices with the multipliers module's code (`io_core.py`, `qcew_tools.py`) and checks the rebuild against `multipliers.csv`: the SLG-pattern Type II output multiplier (2.2613), IO-50 Type II output and IO-50 jobs per $1M. Every difference is 0.0000 (`reproduction_checks` in the model's full output).

### 2.2 San Juan County

The county model applies the Flegg location quotient (FLQ) to the state coefficients.

- FLQ_ij = CILQ_ij × λ, with λ = [log₂(1 + E_r/E_n)]^δ.
- E_r = 6,097 QCEW covered jobs (2024) + 118 Peace Island Medical Center jobs + 30.6 FHL annual-average jobs = 6,245.6. E_n = 3,594,439. λ = 0.224 at δ = 0.25.
- Household purchases are scaled by min(SLQ, 1) (the "SLQ" closure). The FLQ-scaled closure, min(SLQ·λ, 1), is reported as a variant and sets the extreme low (section 2.4).
- The household (labor-income) row inside the closed inverse is multiplied by `SJC_IND_RES`, the share of county industry labor income received by county residents (0.88 / 0.94 / 1.00; WWU CEBR 2025 job flows: 88% of county workers live and work there, about 6% commute in).
- Two QCEW gaps are filled before the location quotients are computed:
  - **IO-47 Hospitals.** The 2024 county QCEW file has no NAICS 622 cell, because PeaceHealth reports off-island. Peace Island Medical Center had 118 employees in July 2022–June 2023 (PeaceHealth FY23 fact sheet, archived copy). They are imputed into county IO-47 and the county total.
  - **FHL.** FHL is probably not in county QCEW (section 7). Its 30.6 annual-average jobs are included in the county total; this changes λ only negligibly.
- **IO-30 Non-store retail** gets county SLQ = 0. Island households' online purchases are not produced in the county. The multipliers module mapped IO-30 to the store-retail quotient (1.371); that mapping is kept in the state run only and is reported as a variant.
- County direct jobs per $ are the state coefficients scaled by the state-to-county average pay ratio (QCEW 2024), as in the multipliers module.

The FLQ δ (0.30 / 0.25 / 0.20) is a separate input, `FLQ_DELTA`. It is **immaterial** for FHL: over its full range it moves county annual output by $0.03M. The county sectors FHL buys from have Type I output multipliers of only 1.02–1.08. The regional uncertainty that matters is the household closure, `SJC_IND_K` (section 2.4).

### 2.3 Jobs coefficients

OFM's 2012 jobs per $1M of output (payroll jobs plus proprietors, annual-average) are converted to 2024 dollars as

e₂₀₂₄ = e₂₀₁₂ / (pay_ratio^(1−w) × CPI_ratio^w),

where pay_ratio is the 2012–2024 change in WA QCEW average pay for the sector's NAICS group, CPI_ratio = 1.366, and w = `JOBS_W` (0 / 0.5 / 1, applied to both regions).

- w = 0 (the multipliers module's central) treats all pay growth as price change. In low-wage sectors, 2012–2024 pay growth was much faster than CPI: 1.84 in IO-50, 1.76 in IO-49 and 2.28 in IO-52, against 1.37. Much of that reflects WA minimum-wage increases, not output prices, so w = 0 understates 2024 jobs per $.
- w = 1 ignores productivity growth and overstates them.
- Central w = 0.5 is the geometric midpoint.

**Validation** (`validations_jobs_per_dollar` in the model's full output). Dean Runyan's 2017p San Juan County figures give 2,320 travel jobs on $237.1M of travel spending: 9.78 jobs per $1M in 2017 dollars, or 7.65 per $1M in 2024 dollars (CPI-U). On the visitor spending mix, the model's county direct jobs per $1M of purchaser spending are 5.62 (w = 0), 6.36 (w = 0.5) and 7.22 (w = 1). The model should sit somewhat below Dean Runyan, because it counts goods at margin (producer prices) while Dean Runyan counts jobs per $ of purchaser spending. The central value is consistent with that; w = 0 looks too low.

### 2.4 Induced effects: calibration per dollar of labor income

In a household-closed model, induced output per $ of Type I labor income is the same for every sector. Call it k: it equals the industry output generated by $1 of household income. The model computes k from the closed inverse as k = Σᵢ L2[i, households]. Induced effects are then scaled by

s = k_target / k_closure.

The scale is defined **per dollar of labor income**, not per dollar of output. A ratio taken per dollar of output depends on the spending pattern's labor share. FHL's labor share is 0.71, while OFM's state and local government pattern has 0.37.

| Region | Closure as built (k) | Calibration target (k_target) | Scale s (central) |
|---|---|---|---|
| Washington | OFM: 1.5291 output per $; 0.4790 induced labor income per $ | **0.7853**: UW research enterprise, IMPLAN (Parker 2024 p. 9): induced output $737.57M ÷ Type I labor income ($736.45M + $202.79M) | 0.5136 |
| San Juan County | SLQ closure at δ = 0.25, r = 0.94: 0.7386 per resident $ (0.751 at r = 1) | **0.4021**: WSF Route 80, San Juan County, IMPLAN (WWU 2024 Table 24): induced output $9.797M ÷ Type I labor income ($24.704M + $1.217M) = 0.3780 per workplace $, ÷ 0.94 resident share = 0.4021 per resident $ | 0.5444 |

The Washington calibration also matches on labor income. Induced labor income per $ of Type I labor income is 0.246 in the model, against 0.248 for UW research (IMPLAN) and 0.243 for WSF statewide (IMPLAN; Type I 1.11, SAM 1.38). OFM as built gives 0.479. A second check comes from transferring the Washington ratio to the county: applying 0.514 to the county SLQ closure gives 0.751 × 0.514 = 0.386, close to the Route 80 value.

**Why OFM overstates induced effects.** OFM's c is PCE bought from WA industries ÷ total WA labor income. It treats all personal consumption as financed by labor income, including PCE paid from transfers and property income. It also has no leakage to income and payroll taxes or to saving. IMPLAN's SAM closure has both. The county SLQ closure inherits the same c.

**Ranges for the closure inputs:**

| Input | Extreme low | Central | Extreme high | Monte Carlo support |
|---|---|---|---|---|
| `WA_IND_K` | 0.7765: WSF statewide, matched on induced labor income (0.2432 ÷ 0.4790 = 0.508 × 1.5291) | 0.7853 | 1.5291: OFM as built | triangular(0.7765, 0.7853, 0.9175); 0.9175 = 0.60 × OFM, an assumption allowing FHL households (pay per job about half of UW research's) a higher consumption share of income |
| `SJC_IND_K` | 0.1933: FLQ-scaled household closure at δ = 0.25 | 0.4021 | 0.751: SLQ closure as built | triangular(0.3086, 0.4021, 0.4956); 0.3086 = Route 80 matched on induced labor income (0.0875 ÷ 0.94 ÷ 0.2228 × 0.7386); the high is symmetric (assumption) |

The uncalibrated closures (OFM as built; county SLQ as built) appear in the extreme high and as the named variant `VARIANT_OFM_CLOSURE_AS_BUILT`. They are kept out of the Monte Carlo because they are known to be biased upward. For the same reason the one-at-a-time sensitivity swings `WA_IND_K` and `SJC_IND_K` over their Monte Carlo support, and the closures as built (and the county FLQ-scaled closure) appear there only as the named variants. The model's induced labor income per $ of Type I labor income in the county (0.121) remains above Route 80's (0.0875). That gap is why the Monte Carlo low uses the labor-income match.

Because the induced level is calibrated per resident dollar, inputs that only reshape the household closure have small effects once s is recomputed. The hospital imputation moves county output by $50, and `SJC_IND_RES` by $0.01M. Without calibration, the hospital imputation would add about $0.16M.

## 3. Equations: analysis by parts

Every spending source becomes a final-demand vector **f** for regional industries, in producer prices. An institution (FHL operations, the research line, UW F&A in the memo line) also has exogenous household income **ℓ**: its payroll paid to residents of the region.

```
x1 = L1 f                       (Type I industry output)
z  = L2 [f ; ℓ]                 (Type II, households closed; county household row × SJC_IND_RES)
x2 = z[industries]
induced = s (x2 − x1),  s = k_target / k_closure
```

That is "Type II minus Type I", applied both to payroll and to supplier labor income, and scaled to the IMPLAN-calibrated level.

**Institution** (operations, research):

- direct output = labor income + purchases, with transfers (scholarships, tuition, stipends) excluded;
- direct jobs, labor income and value added are given;
- indirect = x1;
- induced = s (x2 − x1).

**Industry spending** (students, visitors, construction):

- direct = f;
- indirect = x1 − f;
- induced = s (x2 − x1).

**Measures** for any output vector x:

- output = Σx;
- jobs = e·x / 10⁶ (section 2.3);
- labor income = h·x;
- value added = v·x.

Totals are direct + indirect + induced.

**State and local taxes (method A, headline):**

- Indirect and induced effects, and institutional direct output: T = r_out × output. r_out = 0.0363 (UW research enterprise, IMPLAN, FY2023); high 0.0461 (all UW).
- Visitors' in-county spending: T = r_vis × spending. r_vis = 0.0976, from WA State Tourism figures via WWU Table 4 ($29.20M ÷ $299.08M). The WWU table header dates them 2023 and the WWU text 2019. Low = 0.0962 (Dean Runyan 2017p: $22.8M ÷ $237.1M; 2016 revised gives 0.0975). r_vis includes lodging taxes and is applied to in-county visitor spending only.
- Students' in-county spending: T = 8.5% × 0.827 × spending. 8.5% is the Friday Harbor combined sales tax rate (FHL 2026 rates page); 0.827 is the taxable share of the student spending mix (food service, retail and arts/recreation). Groceries and motor fuel are exempt from retail sales tax (RCW 82.08.0293, 82.08.0255). Students' meals and beds are paid to FHL, so no lodging tax applies.
- Washington travel outside the county: T = r_out × non-ferry spending. WSF fares are fare revenue, not tax (ferry share 0.165).
- County taxes are **state and local taxes generated by activity in San Juan County, mostly state revenue**. They are not county government revenue.

**Cross-check (method B):** T = ITEP household rate (9.4% / 10.9% / 13.8%) × 0.759 × resident labor income + the direct taxes on user spending above. 0.759 is the wage share of FHL labor income. Method B gives $0.42M for Washington and $0.33M for the county, against $0.36M and $0.27M from method A (central).

## 4. Spending sources and how each enters

### 4.1 Operations

Source: the operations module, a typical FY2023–FY2025 year in 2024 dollars.

- **Labor income** $2.90M (P10–P90 $2.55–3.30M).
- **Purchases** $1.20M. In the extreme bounds and sensitivity the low is the itemized floor, $0.79M, which the module instructs to use instead of its P10 of $0.50M; the high is $2.10M. In the Monte Carlo, purchases are not drawn directly (section 6).
- **Direct output** = labor income + purchases = **$4.10M**. The module's $4.30M headline also contains $0.21M of scholarships, which are excluded as a transfer: they mostly pay FHL fees, which are already FHL revenue.
- **Direct jobs: 30.6 annual-average jobs** (21.3–41.3), on a 12-month FTE basis: year-round FTE (R01–R14) + seasonal persons × months/12 (R15) + summer instructors × months/12 (R16 × 9/12, converting the module's 9-month FTE) + teaching faculty (R17) + TA-quarters/4 (R18) + FHL postdocs (R19). This matches UW's EIR convention ("direct jobs based on UW FTEs"), the annual-average concept of the OFM coefficients used for indirect and induced jobs, and QCEW's annual average of monthly jobs. The module's headcount, 49 people employed during the year (37–65), is a memo line only.
- **Direct value added** = labor income (central and low), the usual IMPLAN analysis-by-parts treatment of an institution. The high case uses the OFM SLG ratio, 1.214 × labor income, which adds capital consumption.

**Payroll paid to residents:**

- San Juan County: 87.4% (85.8–88.8%, from the module).
- Washington: **96.1%** (94.3–97.9%), computed as 1 − (0.0691 × `INSTR_OOS` + 0.0414 × `TA_OOS`).
  - 0.0691 and 0.0414 are the roster labor-income shares of summer instructors (R16: 1.667 9-month FTE × $94,786 × 1.226) and TAs (R18: 2.0 × $49,166 × 1.182), out of the bottom-up $2.80M.
  - `INSTR_OOS` (0.7 / 0.5 / 0.3) and `TA_OOS` (0.2 / 0.1 / 0) are the non-Washington shares (assumptions).

**Purchases** are split into the module's six itemized categories, weighted by its central amounts: utilities 0.279, food 0.134, maintenance 0.264, vessels 0.075, scientific supplies 0.112, IT/administration/other 0.136. Each category then receives:

1. **Geography.** The module's gross shares bought in the county and in Washington, by category.
2. **Services vs goods.** A services share (`SVC_*`) goes to service sectors in full:
   - utilities: electric IO-06 (`SPLIT_UTIL_ELEC` 0.6; 0.4–0.8); water and sewer IO-08 and solid waste IO-52 split the rest (`SPLIT_UTIL_WATER` 0.625; 0.5–0.75);
   - maintenance: IO-10, repair construction;
   - vessels: IO-36 support activities (`SPLIT_VESSEL_IO36` 0.5; 0.3–0.7) and the rest to IO-25 boat repair;
   - IT/administration/other: a blend (`MIX_IT` 0.5; 0–1) of mix A (IO-37 software/ISP 0.25, IO-41 insurance 0.20, IO-38 telecom 0.20, IO-43 professional 0.15, IO-35 freight/postal 0.10, IO-33 ferries 0.05, IO-15 printing 0.05) and mix B (0.10, 0.05, 0.30, 0.15, 0.20, 0.10, 0.10). Staff air travel and out-of-state hotels fall in the share bought outside Washington.
3. **Margins on goods.** Goods bought from county retailers place only the retail margin in IO-31. In the state run they also add the wholesale margin on the remaining value to IO-29. Goods bought elsewhere in Washington add only the wholesale margin. The producer value is treated as imported in both regions.

| Goods | Retail margin (ARTS 2022) | Wholesale margin (AWTS 2022) |
|---|---|---|
| Food | 0.280 (grocery 4451) | 0.162 (4244) |
| Maintenance materials | 0.360 (4441) | 0.323 (4237) |
| Marine fuel (`VESSEL_FUEL_MARGIN`) | 0.186 (gasoline 447, 2021 since 2022 is suppressed); low 0.161; high 0.329 (all retail, for a low-volume fuel dock) | 0.041 (4247) |
| Scientific supplies, office goods | 0.329 (all retail excl. motor vehicles) | 0.241 (4246), 0.239 (4241) |

Low and high margins are the 2018–2022 minimum and maximum. The resulting central final demand (`final_demand_central` in the model's full output) is $0.50M in San Juan County (utilities $0.32M, maintenance contracts $0.08M, retail margins $0.04M) and $0.73M in Washington.

### 4.2 Research (separate line, counted once)

Research follows the research module's recommendation: one separate line, direct costs only, facility awards excluded.

- **Gross award dollars at FHL,** average FY2022–24: low $402,630 (core awards only, a documented floor); central **$606,080**; high $809,530.
  - The high adds partial awards × 0.25 and resident-PI awards × 0.75.
  - The central adds half of that add-on (assumption).
  - Check: FY2019 is the one year with a published FHL figure (UW Office of Research: $324,049 direct + $50,312 indirect). It implies an add-on weight of 0.88 on the FY2019 series, so 0.5 is conservative. A triangular distribution with an interior mode then puts the central inside its own Monte Carlo range: $0.67M [0.56–0.87] of Washington total output.
- **Direct-cost share:** 0.865606 (unrounded; 324,049 ÷ 374,361), range 0.73–1.00. Central direct costs are $524,626. The research module's core-only line ($348,519) equals the low gross × 0.865606.
- **F&A** (central $81,454 a year) is spent by UW in Seattle. It is a **memo line in the state run only** (`memo_research_FA_uw_seattle_not_in_totals`: $0.15M output, 0.6 jobs), not in the totals. UW returns part of F&A to units, so it may already finance part of FHL's core or other UW support.
- **Spending pattern** (FHL department, FY2019): salaries 55.6%, supplies and services 34.3%, tuition and stipends 10.1% (excluded as transfers).
- **FHL fees in research purchases** (`RES_FHL_FEES`). Supplies and services include payments back to FHL for room, board, lab and boat fees, which are already FHL operations revenue (C_SELFSUST). They are removed: REU participants 840 nights × $62.29 a day (the summer course housing-and-dining rate, $2,309 ÷ 35 days in 2026, deflated) + $10,000 of other awards' fees = $62,321 central. The range runs from $77,321 down to 0; the 0 case allows that REU participant costs are booked under stipends, which are already excluded.
- **Research purchases** = $524,626 × 0.3432 − $62,321 = **$117,735**.
  - A services share `RES_SVC_SHARE` (0.4; 0.2–0.6) goes to IO-44 (R&D, testing and analytical services), bought 10% in the county (5–20%) and 50% in Washington (30–70%).
  - The rest is scientific supplies, at the operations science-supply geography and margins.
- **Direct jobs** = research labor income ÷ pay per job, all including benefits:
  - central $70,952 (NIH NRSA postdoc stipend $61,008 × 1.163);
  - fewer-jobs case $94,824 (FHL operations labor income per annual-average job);
  - more-jobs case $58,114 (graduate RA, OEWS 25-9044 WA median × 1.182).
- **Research payroll paid to residents** (assumption): county 0.5 / 0.7 / 0.9; Washington 0.9 / 0.95 / 1.0.

Research direct output is $0.41M and research direct jobs are 4.1. Grant-paid people are outside the operations roster, as the operations module specifies, so payroll is not counted twice.

### 4.3 Students and visitors

Source: the users module, calendar 2024. It counts only off-FHL spending by people from outside the county. Room, board and fees paid to FHL are inside operations, and tuition is excluded.

- **Students:** summer course students, spring and autumn quarter students, MARBIO 488 and REU interns.
- **Visitors:** workshop participants, visiting researchers, visiting instructors, Whiteley scholars, and conference and field-trip groups.

| Line (central) | San Juan County run | Washington run, contribution | Washington run, new money |
|---|---|---|---|
| Students | $195K (in county) | $231K (in county + travel) | $59K (out-of-state, non-UW-registered) |
| Visitors | $306K (in county) | $399K | $240K |

In the Washington contribution, visiting instructors' spending is multiplied by `INSTR_OOS`. The pay of Washington-resident instructors already induces Washington household spending through the operations line, so counting their spending again would double count it. In the county run, instructors' spending is kept in full, because their pay is mostly excluded from county induced effects (resident share 0–30% in the operations module).

**Sector mapping:**

- Students use the San Juan County visitor non-lodging mix (Dean Runyan 2017, $M): food service 88.6, food stores 17.0, local transport and gas 13.7, arts and recreation 29.0, retail 29.5.
- Visitors use the users module's central commodity split, minus the student part: food service 0.288, food stores 0.224, local transport and gas 0.047, arts and recreation 0.099, retail 0.100, accommodation 0.242.
- Food service → IO-50. Accommodation and arts/recreation → IO-49 (OFM combines them in one sector). Food stores and retail → the retail margin in IO-31, plus the wholesale margin in IO-29 in the state run. Local transport and gas → fuel (`FUEL_SHARE` 0.5; 0.3–0.7) at the gasoline-station margin, with the rest to IO-35.
- Washington travel outside the county (state run only): ferry 0.165 → IO-33 (WSF is in OFM water transportation); airport shuttle and ground 0.338 → IO-35; travel-day meals 0.366 → IO-50; Seattle lodging 0.131 → IO-49.

The commodity shares are the central shares in every scenario. All of these mappings are listed in `model_inputs.csv` (`MAP_SECTORS`).

### 4.4 Construction (one-time, reported separately)

- **Spending:** construction 2015–2024 totals $2.19M ($2.06–2.32M, research_construction module). It covers the 2015 Fernald basement remodel, the 2019 sewer lift station, the **construction part (2/3) of NSF 2218256**, and an FHL share of state minor works.
- **Equipment** — the remaining 1/3 of NSF 2218256, $117,483 in 2022–24 — is excluded from every line and shown as the memo input `EQUIP_NSF`. It is one-time equipment, mostly bought out of state.
- **Treatment:** the spending enters IO-10.
  - `CONSTR_SJC_SHARE` (0.3 / 0.5 / 0.7) is the share of contract value paid to **county-based contractors, gross of their own purchases**. Leakage of mainland materials is left to the FLQ-regionalized IO-10 column, so it is not removed twice.
  - Washington share 0.90 / 0.95 / 1.00.
- **Units:** total jobs from the decade's spending are job-years. The annual average is the decade total ÷ 10.
- **Exclusions:** the three-year capital average ($160K a year for 2022–24) is **not** included in operations; using both would count 2022–24 twice. R/V Kittiwake (no published price) and the dock replacement (unverifiable $21M) are also excluded.

## 5. Contribution and new-money impact

The model follows the NERRS and Hatfield practice. The **contribution** counts all FHL-related activity in the region. The **new-money impact** counts only spending financed from outside the region, which would not occur there without FHL.

One donor share, `DONOR_SHARE` (0.24; 0.22–0.25, FHL's "almost 25%" of the money it spends each year, an estimate published in March 2023; range an assumption), is used in both regions. The revenue-side composition's 0.256 is a memo line. The input is direction-neutral: a higher value lowers county new money and raises state new money.

**San Juan County:**

- Operations: 1 − donor share (0.24) × island-donor share (0.3 / 0.2 / 0.1) − local fee share (0.02 / 0.01 / 0) = **0.942** (0.908–0.976).
- Research: federal money, 1.0.
- Students and visitors: 1.0, because the users module counts only people from outside the county.
- Construction: 1.0 (UW, state and NSF money).

**Washington:**

- Operations: self-sustaining revenue share (0.2612) × out-of-state share of user spending (0.494; 0.305–0.642) + donor share (0.24) × out-of-state donor share (0.5; 0.3–0.7) = **0.249** (0.152–0.336). Core state and UW funds and other UW money are transfers within the state.
- Research: 1.0. Every award in the line comes from NSF, NIH or USAspending records.
- Students and visitors: only out-of-state spending counts. UW-registered spring, autumn and MARBIO 488 students are set to zero, because they would otherwise spend in Seattle.
- Construction: only the NSF-funded part, $234,965 of $2.19M.

## 6. Ranges

**Reported range: Monte Carlo P10–P90.** 10,000 draws, seed 20261102.

- Triangular (mc_low, central, mc_high) for assumption ranges. mc_low and mc_high equal the extreme bounds except for the two closure inputs (section 2.4).
- Two-piece normal fitted to P10 / P50 / P90 where a module reported percentiles: operations labor income, operations total, county resident share of payroll, and the users-module totals.
- **Operations purchases** = OPS_TOTAL draw (3.6 / 4.3 / 5.6M) − OPS_LI draw − scholarship draw, floored at the itemized $792,828. This is how the operations module itself builds purchases.
  - 27.0% of draws sit at the floor (the module reports 24% below it).
  - Direct operations output (labor income + purchases) has P10–P90 of $3.65–5.39M, against about $3.39–5.39M for the module's total minus scholarships. The difference is the floor.
  - The module's 3.6M and 5.6M are the lowest P10 and highest P90 of its three routes, an envelope, so the upper tail of purchases (P90 $2.52M) is wider than the module's own pooled P90 ($2.1M).
- **User spending:** one uniform draw. In-county amounts are scaled by the users-module county draw ÷ its median, and travel amounts by the Washington draw ÷ its median. The median of each therefore equals its central value.
- **Correlation.** Inputs are otherwise independent. Correlated errors would widen the range: for example, a common bias in the users module's day counts, or in the operations roster and the purchase categories. Labor income and purchases are not an example: given the total they are negatively correlated, and the residual construction already captures that.

The central lies inside P10–P90 for every annual headline total and for the operations, research, student and visitor lines. Where `TAX_OUT` or `VA_LI_INST` dominate, the central can sit just below P10: construction taxes (135,981 against a P10 of 137,392), induced taxes, and the resident-faculty memo totals. Several measures are right-skewed, because some triangular inputs have their mode at one end (`TAX_OUT`, `VA_LI_INST`, `RES_FHL_FEES`) and the purchases residual has a long upper tail. Their P50 is therefore above central: Washington output has central $9.11M against a P50 of $9.41M.

**Extreme bounds (appendix only).** All-low takes every `bounds` input at its impact-reducing value, including the WSF-matched Washington closure, the FLQ-household county closure, δ = 0.30 and pay-adjusted jobs coefficients. All-high takes the mirror image, including the closures as built and CPI-adjusted jobs coefficients. `neutral` inputs (utility, vessel and IT splits; the donor share) and `fixed` inputs are held at central. The bounds compound every extreme, so they are not a likely range.

**One-at-a-time sensitivity** (`model_sensitivity.csv`). Each input group is set to its low and then its high value, with everything else at central. The two household-closure inputs, `WA_IND_K` and `SJC_IND_K`, are set to the ends of their Monte Carlo support instead, because their extreme bounds are the uncalibrated closures (section 2.4). Four named model variants are rows of their own ("low" = central, "high" = variant): two carry the uncalibrated household closures (section 2.4), and two test county data choices (the IO-30 mapping, and QCEW without the hospital imputation). The table reports the swing in annual total output, jobs and labor income for both regions and bases, and in construction job-years. Rows are sorted by the county output swing and carry a rank column for each metric.

## 7. Results (2024 dollars)

Format: **central [Monte Carlo P10–P90]**.

**Annual contribution, typical FY2024:**

| Region | Output | Jobs | Labor income | Value added | State + local taxes |
|---|---|---|---|---|---|
| Washington | $9.11M [8.37–11.89] | 54.4 [50.0–62.9] | $4.63M [4.30–5.28] | $5.42M [5.25–6.54] | $0.36M [0.35–0.50] |
| San Juan County | $6.67M [6.01–8.61] | 47.9 [42.9–54.9] | $3.88M [3.54–4.36] | $4.27M [4.09–5.11] | $0.27M [0.26–0.37] |

**Annual new-money impact:**

| Region | Output | Jobs | Labor income | Value added | Taxes |
|---|---|---|---|---|---|
| Washington | $2.97M [2.66–3.75] | 18.7 [16.4–21.5] | $1.50M [1.33–1.73] | $1.77M [1.64–2.15] | $0.12M [0.12–0.16] |
| San Juan County | $6.34M [5.71–8.16] | 45.6 [40.8–52.2] | $3.68M [3.36–4.14] | $4.05M [3.88–4.86] | $0.26M [0.25–0.35] |

**Annual contribution by effect (central):**

| Region | Effect | Output | Jobs | Labor income | Value added | Taxes |
|---|---|---|---|---|---|---|
| Washington | direct | $5.04M | 38.4 | $3.37M | $3.47M | $0.21M |
| Washington | indirect | $1.23M | 3.6 | $0.37M | $0.57M | $0.04M |
| Washington | induced | $2.84M | 12.4 | $0.89M | $1.38M | $0.10M |
| San Juan County | direct | $4.89M | 37.9 | $3.32M | $3.40M | $0.21M |
| San Juan County | indirect | $0.56M | 1.9 | $0.17M | $0.26M | $0.02M |
| San Juan County | induced | $1.21M | 8.0 | $0.39M | $0.61M | $0.04M |

**Annual totals by source (central total output [P10–P90]; jobs):**

| Source | Washington contribution | Washington new money | San Juan County contribution | San Juan County new money |
|---|---|---|---|---|
| Operations | $7.52M [6.76–10.25]; 44.0 | $1.87M; 10.9 | $5.71M [5.05–7.62]; 39.5 | $5.38M; 37.2 |
| Research | $0.67M [0.56–0.87]; 5.3 | $0.67M; 5.3 | $0.50M [0.40–0.64]; 4.7 | $0.50M; 4.7 |
| Students | $0.34M [0.27–0.42]; 1.9 | $0.09M; 0.5 | $0.18M [0.14–0.23]; 1.5 | $0.18M; 1.5 |
| Visitors | $0.57M [0.46–0.71]; 3.3 | $0.34M; 2.0 | $0.27M [0.21–0.34]; 2.2 | $0.27M; 2.2 |

**Construction 2015–2024 (one-time):**

| Region | Output | Job-years |
|---|---|---|
| Washington, contribution | $3.75M [3.62–3.94] | 12.3 [11.6–13.3] |
| Washington, new money (NSF part) | $0.40M | 1.3 |
| San Juan County | $1.25M [0.97–1.52] | 5.4 [4.2–6.6] |

That is about 1.2 jobs a year in Washington and 0.5 in the county.

**Memo lines (not in totals):**

- UW F&A on FHL research, Washington: $0.15M output, 0.6 jobs.
- UW departmental payroll for resident faculty at FHL: Washington $0.92M output, 5.8 jobs; county $0.70M output, 5.2 jobs.

**Model variants** (central inputs; annual contribution output and jobs):

| Variant | Washington | San Juan County |
|---|---|---|
| Closures as built (WA OFM, county SLQ; s = 1) | $11.79M; 66.2 | $7.68M; 54.6 |
| County FLQ-household closure, unscaled | — | $6.03M; 43.7 |
| County QCEW as published (no hospital imputation) | — | $6.67M; 48.1 |
| County IO-30 as in the multipliers module | — | $6.67M; 47.9 |

**Extreme bounds (all-low – all-high), annual contribution:**

- Washington: output $6.30–19.15M; jobs 33.3–114.2; labor income $3.57–7.84M; value added $4.05–10.91M; taxes $0.23–0.97M.
- San Juan County: output $4.30–12.02M; jobs 26.9–91.5; labor income $2.96–5.64M; value added $3.09–7.52M; taxes $0.16–0.66M.

New money:

- Washington: output $1.23–8.75M; jobs 7.0–54.0.
- San Juan County: output $3.93–11.80M; jobs 24.6–90.0.

### Context ratios

Denominators: QCEW 2024 San Juan County (6,097 covered jobs, $324.2M wages, $53,170 average pay per job); BEA CAEMP25N 2022 (11,979 total jobs including 5,306 proprietors; 2022 is the latest year in BEA's public bulk file); BEA CAINC4 2024 earnings by place of work ($670.9M, compensation plus proprietors' income). Each ratio states its job concept.

- **FHL direct jobs (30.6 annual-average FTE) are 0.50% of county jobs** (0.35–0.67%): FTE ÷ (QCEW + FHL). FHL is included in the denominator because QCEW probably does not contain it (below). As a memo, FHL's headcount of 49 people over the year is 0.80%; it counts seasonal and part-year people whole.
- **FHL wages are 0.67% of county covered wages** (0.60–0.77%), with FHL included in the denominator.
- **Pay.** FHL wages per annual-average job are $71,935, or 1.35 × the county average pay per job (range 0.89–2.20 ×). Annual-average jobs are the QCEW concept: each seasonal or part-year job counts for the months it is filled. Per headcount, FHL wages are $44,898, or 0.84 ×, which is not like-for-like because seasonal staff, part-year instructors and TAs count as whole jobs.
- **All FHL-related jobs in the county** (direct + indirect + induced, including proprietors through the OFM coefficients) are **0.40%** of BEA total employment including proprietors (0.22–0.76%).
- **All FHL-related labor income in the county** is 0.58% of BEA earnings by place of work (0.44–0.84%).
- **QCEW coverage of FHL.** The only public measured figure is the 2022 QCEW cell for state colleges and universities in the county (NAICS 61131, 2 establishments, probably FHL plus WSU Extension): 23 jobs and $942K of wages, which is $1.01M in 2024 dollars ($43,902 per job). That is well below the modeled $2.20M of FHL wages. QCEW therefore never fully recorded FHL's in-county payroll, even before 2023. The 2014–2021 cells are suppressed; county state-government employment fell from 100 (2014) to 65 (2022); and the 2022–2023 reporting change (23 → 6 jobs) reduced the recorded figure further. Treat the 2022 cell as a measured lower bound, not as FHL's payroll.

### Largest one-at-a-time swings (annual contribution output)

| Rank | San Juan County | Swing | Washington | Swing |
|---|---|---|---|---|
| 1 | Operations purchases | $1.95M | Variant: closures as built | $2.69M |
| 2 | Variant: closures as built | $1.02M | Operations purchases | $2.65M |
| 3 | Operations labor income | $1.01M | Operations labor income | $1.32M |
| 4 | Visiting researchers | $0.66M | Visiting researchers | $1.14M |
| 5 | Variant: FLQ-household closure | $0.63M | Conference and field-trip groups | $1.14M |
| 6 | County induced calibration `SJC_IND_K` (Monte Carlo support) | $0.56M | Research award dollars | $0.51M |
| 7 | Research award dollars | $0.38M | Washington induced calibration `WA_IND_K` (Monte Carlo support) | $0.51M |

- Among the inputs, operations purchases lead in both regions. Household-spending effects matter mainly through the choice of closure: the closures-as-built variant moves output by $2.69M in Washington and $1.02M in the county, against $0.51M and $0.56M for the calibrated inputs over their Monte Carlo support.
- The FLQ δ ranks 21st for the county ($0.03M). `SJC_IND_RES` is $0.01M and the hospital imputation $50, because the calibration absorbs them (section 2.4).
- For jobs, direct FHL jobs (`OPS_JOBS`) lead in both regions with a 20.0-job swing, followed by the closures-as-built variant (Washington 11.8 jobs, county 6.7). Over their Monte Carlo support the induced calibrations move Washington jobs by 2.2 and county jobs by 3.7. `JOBS_W` moves Washington jobs by 4.3 and county jobs by 3.0.
- For Washington new money, the largest inputs are the out-of-state donor share ($0.72M) and the out-of-state user share ($0.66M).

## 8. Cross-checks and peer comparison (central; `cross_checks` in the model's full output)

**Per dollar of labor income.** This is the comparison that is valid across spending patterns.

| Quantity | Model | Benchmark |
|---|---|---|
| WA induced output per $ of Type I labor income | 0.785 | UW research IMPLAN 0.785 (calibration target); OFM as built 1.529 |
| WA induced labor income per $ of Type I labor income | 0.246 | UW research IMPLAN 0.248; WSF statewide IMPLAN 0.243; OFM 0.479 |
| County induced output per $ of resident income | 0.402 | Route 80 IMPLAN 0.378 per workplace $ (0.402 per resident $); SLQ closure as built 0.739 |
| County induced labor income per $ of Type I labor income | 0.121 | Route 80 IMPLAN 0.0875 (Monte Carlo low matches it) |

**Total ÷ direct ratios, by job concept:**

| | Output | Jobs | Direct-job concept |
|---|---|---|---|
| FHL operations, Washington | 1.83 | 1.44 | annual-average FTE |
| FHL all annual activity, Washington | 1.81 | 1.42 | FTE + industry jobs |
| FHL operations, San Juan County | 1.39 | 1.29 | FTE |
| FHL all annual activity, San Juan County | 1.36 | 1.26 | FTE + industry jobs |
| UW FY23 operations (IMPLAN, state) | 2.10 | 1.83 | UW FTE |
| UW research enterprise (IMPLAN, state) | 1.99 | 2.01 | UW jobs |
| Bigelow Laboratory 2024 (RIMS II, Maine) | 1.83 | 2.43 | 111 jobs |
| Carteret County 8 marine agencies 2017 (county) | 1.53 | 1.96 | FTE |
| Duke Marine Lab 2017 (county) | 1.53 | 2.07 | employment |

**Output.** The Washington output ratio (1.83) sits below UW's own operations ratio (2.10), which includes capital spending with a large supply chain, and close to Bigelow. The county ratio (1.36–1.39) is below the mainland counties of Carteret and Duke (1.53). That is expected for an 18,700-person island county where most goods are imported and the calibrated induced effect is about half of what OFM implies.

**Jobs.** FHL's jobs ratio is lower than the peers' for a measurable reason. FHL has many direct jobs per dollar of payroll: 11.4 direct jobs per $1M of direct labor income, against 7.2 for UW research. Its pay per job is lower, and it has seasonal, dining and housekeeping staff. Induced jobs per dollar of labor income are similar to UW's: in Washington, 3.32 induced jobs per $1M of Type I labor income, against 3.53 for UW research. The same supported jobs are therefore divided by a larger direct base.

**User spending.** Total county output per $ of user purchaser spending is 0.91, below the published 1.36–1.42. The published multipliers apply to purchaser spending with goods at full value (and RCO is an IMPLAN multi-region model). This model counts only the retail margin on goods, at producer prices, and its induced effect is calibrated down.

## 9. Assumptions (all in `model_inputs.csv`, each with a range and a reason)

**The model's own assumptions:**

- **Operations:**
  - Washington resident share of payroll, through `INSTR_OOS` and `TA_OOS`;
  - value added per $ of institutional labor income (`VA_LI_INST`);
  - services shares `SVC_MAINT`, `SVC_VESSEL`, `SVC_IT`;
  - splits `SPLIT_UTIL_ELEC`, `SPLIT_UTIL_WATER`, `SPLIT_VESSEL_IO36`, `MIX_IT` (with mixes `MAP_IT_MIX_A/B`);
  - marine-fuel margin `VESSEL_FUEL_MARGIN`;
  - fixed sector mappings `MAP_SECTORS`.
- **Users:** fuel share of local transport `FUEL_SHARE`; central commodity shares used in every scenario; instructors' spending in the Washington contribution scaled by `INSTR_OOS`.
- **Research:**
  - central inclusion weight of partial and resident-PI awards (inside `RES_GROSS`);
  - `RES_FHL_FEES`, `RES_SVC_SHARE`, `RES_SVC_GC`, `RES_SVC_GW`;
  - pay per research job `RES_COMP`;
  - residence shares `RES_LI_RES_SJC`, `RES_LI_RES_WA`.
- **Regional model:**
  - induced calibration `WA_IND_K` and `SJC_IND_K`, with their Monte Carlo highs;
  - `FLQ_DELTA`, `SJC_IND_RES`, `JOBS_W`;
  - hospital and FHL imputations (measured counts; their use is a model choice); county IO-30 LQ = 0.
- **Taxes:** student taxable share (`STU_TAXABLE`, from the Dean Runyan mix and RCW exemptions); application of `TAX_OUT` to travel outside the county.
- **Construction:** `CONSTR_SJC_SHARE` (contract value to county contractors) and `CONSTR_WA_SHARE`.
- **New money:** `ISLAND_DONOR_SHARE`, `LOCAL_FEE_SHARE`, `OOS_DONOR_SHARE`; one donor share for both regions.

**Assumptions inherited from the input modules,** listed there with their sources:

- operations: roster FTE, core funding, purchases by category and their geography;
- users: headcounts, days, daily spending and out-of-state shares;
- research: shares for partial and resident-PI awards; the direct-cost share;
- multipliers: the FLQ method and the jobs-coefficient update.

## 10. Limitations

- **Old structure.** The 2012 OFM table is the latest Washington I-O model, and ratio multipliers assume its production structure holds in 2024. FLQ is a non-survey regionalization, and San Juan County (0.17% of state employment) is smaller than typical FLQ calibration regions.
- **Closure.** OFM's Type II closes only households, on labor income: c = PCE ÷ labor income, with no tax, saving or transfer-income leakage. **It overstates induced effects relative to IMPLAN SAM by about 2× per dollar of labor income in Washington** (1.529 vs 0.785), and the county SLQ closure by about 1.8×. The model calibrates both to IMPLAN benchmarks per dollar of labor income. The county rests on a single benchmark, WSF Route 80. Route 80 crews may partly live off-island, which would depress the benchmark, so the county Monte Carlo is wider than the state's.
- **Unpublished FHL inputs.** FHL publishes no budget, payroll or vendor list. Operations rests on the operations module's three-route reconciliation. Purchases are the largest single input sensitivity in both regions.
- **Category mapping.** Purchase categories are mapped to sectors and split into services and goods by assumption. The sector splits within a category (utilities, vessel services) are neutral inputs, each moving annual output by under $0.01M; the services shares of maintenance, vessel, IT and research purchases are bounded inputs, each moving Washington output by under $0.07M.
- **Users.** No survey of FHL users exists. Visiting researchers make up 41% of county user spending, and their counts, days and meal-plan shares are partly assumed. The 2017 Dean Runyan mix is a fixed composition.
- **Research.** The central inclusion weight (0.5) is an assumption; FY2019 data imply a higher weight. NIH and pre-API award timing is spread evenly.
- **Taxes.** The rates are ratio approximations. The UW rate is an IMPLAN state and local tax total per $ of output. It is applied to county results as taxes generated by activity in the county, not as county revenue. FHL itself pays no B&O or property tax on its own direct output; the ratio represents UW's average.
- **Pay residence.** Commuting is in the county closure through `SJC_IND_RES` (0.88–1.00) and the IMPLAN calibration. Seasonal non-resident workers in accommodation and food service may push the true share toward the low end.
- **Producer value of goods** is treated as imported in both regions. This understates Washington effects slightly, because the state has food processing and a refinery.
- **Monte Carlo independence.** Except for the operations residual and the user-spending scaling, inputs are independent. Common biases (for example, in the users module's day counts or across the operations roster) would widen the range.
- **BEA vintage.** County total employment including proprietors is 2022, the latest year in BEA's public bulk file; earnings are 2024.
- **Exclusions:**
  - downstream innovation value (aequorin/GFP and others; section VIII of the report);
  - research grants held at visitors' home institutions;
  - catalytic and ecosystem-service value;
  - acknowledged-award totals (Fig. 22 of the report), which are not FHL spending;
  - UW departmental payroll for resident faculty and research F&A, both shown only as memo lines;
  - the equipment part of NSF 2218256.

## 11. Double-counting rules applied

- **Operations:** room, board and fees paid to FHL are inside operations. Users count off-FHL spending only. Scholarships are excluded.
- **Research:**
  - a separate line, counted once;
  - grant-paid staff are outside the operations roster;
  - FHL fees paid from research awards (REU room and board, lab and boat fees) are removed from research purchases, because they are inside operations revenue;
  - facility awards are outside research; their construction part is in construction and their equipment part is excluded;
  - F&A is a memo line, because UW may return part of it to FHL's core or other UW support.
- **Instructors:** in the Washington contribution, only out-of-state instructors' spending stays in the visitors line. Washington-resident instructors' pay already induces Washington spending.
- **Capital:** reported once, as one-time construction. The three-year capital average is not included in operations.
- **Excluded totals:** finance_items rows are not summed. Acknowledged-award totals are not used as spending. UW statewide IMPLAN totals are not pro-rated.

## 12. Sources

**Input modules.** These files of the census pipeline (not published) carry the original URLs for each item.

- `model/multipliers/multipliers.csv`, `build_multipliers.py`, `io_core.py`, `qcew_tools.py` and `raw/` (OFM table, QCEW files).
- `model/operations/operations_inputs.csv` and `notes.md`.
- `model/users/user_inputs.csv` and `build_user_inputs.py`.
- `model/research_construction/research_summary.csv`, `research_by_uw_fy.csv`, `capital_summary.csv`, `capital_by_year.csv` and `notes.md`.
- `people_and_local/fhl_rates_2026.csv` (FHL 2026 rates and the 8.5% sales tax; Internet Archive copies of fhl.uw.edu pages) and `san_juan_county_context.csv`.

**Input-output model:**

- OFM, 2012 Washington Input-Output Model: <https://ofm.wa.gov/wp-content/uploads/sites/default/files/public/dataresearch/economy/I-O_2012_table.xlsx>
  - worksheets: <https://ofm.wa.gov/wp-content/uploads/sites/default/files/public/dataresearch/economy/I-O_2012_impact_worksheets.xlsx>
  - report: <https://ofm.wa.gov/wp-content/uploads/sites/default/files/public/dataresearch/economy/IO_2012_report.pdf>
- BLS QCEW, area files 2012 and 2024 for WA (53000) and San Juan County (53055): <https://data.bls.gov/cew/data/api/2024/a/area/53055.csv> and <https://data.bls.gov/cew/data/api/2024/a/area/53000.csv>.
- FLQ method: Flegg, Webber & Elliott (1995), *Regional Studies* 29(6):547–561; Flegg & Webber (1997), *Regional Studies* 31(8):795–805; Flegg & Webber (2000), *Regional Studies* 34(6):563–569; Flegg & Tohmo (2013), *Regional Studies* 47(5):703–721.

**Calibration benchmarks and tax ratios:**

- Parker Strategy Group, *UW Economic Impact Report* (FY2023, published 2024), pp. 6–11. UW operations ratios, the research enterprise table (output, jobs, labor income, value added) and state and local taxes: <https://cdn.uw.edu/wp-content/uploads/sites/18/2024/10/29155253/PSG-UW-Economic-Impact-Report.pdf>
- WWU CEBR, *Washington State Ferry System Economic Impact Study* (Dec 2024): Tables 1–2 (statewide multipliers), Table 4 (San Juan County travel taxes), Table 24 (Route 80 in San Juan County). <https://cbe.wwu.edu/sites/cbe.wwu.edu/files/2025-01/WWU%20Ferry%20Study%20Full%20Report.pdf>
- WWU CEBR, San Juan County profile 2025 (job flows, seasonality): <https://cbe.wwu.edu/sites/cbe.wwu.edu/files/2025-10/San%20Juan%20profile_2025_FINAL.pdf>
- Earth Economics for RCO (2025), San Juan County recreation multiplier 1.42: <https://rco.wa.gov/wp-content/uploads/2025/11/EconomicReportOutdoorRecreation2025.pdf>
- ITEP, *Who Pays?* 7th edition, Washington: <https://itep.org/whopays/washington-who-pays-7th-edition/>
- Dean Runyan Associates, *Washington State County Travel Impacts 2000–2017p*, San Juan County pp. 127–129: spending mix, jobs, taxes. <https://bremertonwa.gov/DocumentCenter/View/7763/7-PDF>.
- RCW 82.08.0293 (food exemption) and RCW 82.08.0255 (motor fuel exemption).

**County data:**

- BEA Regional, CAEMP25N (total full- and part-time employment by industry, 2001–2022): <https://apps.bea.gov/regional/zip/CAEMP25N.zip>
- BEA Regional, CAINC4 (personal income and earnings by component, 1969–2024): <https://apps.bea.gov/regional/zip/CAINC4.zip> (CAINC30 was also downloaded and not used, because its employment lines are absent in this release).
- PeaceHealth Peace Island Medical Center FY23 fact sheet (118 employees): <https://www.peacehealth.org/sites/default/files/2023-11/FY23%20PeaceHealth%20Peace%20Island%20Fact%20Sheet.pdf>, read from <https://web.archive.org/web/20240527090856/https://www.peacehealth.org/sites/default/files/2023-11/FY23%20PeaceHealth%20Peace%20Island%20Fact%20Sheet.pdf> (the live site returned a bot check).

**Margins:**

- U.S. Census Bureau, Annual Retail Trade Survey 2022 (benchmarked), gross margin as a percentage of sales: <https://www2.census.gov/programs-surveys/arts/tables/2022benchmarked/gmper.xlsx>
- U.S. Census Bureau, Annual Wholesale Trade Survey 2022 (revised), Table 4: <https://www2.census.gov/programs-surveys/awts/tables/2022revised/2022_awts_purchmarg_nomsbo_table4.xlsx>

**Research spending pattern and pay:**

- UW Office of Research, FY2019 annual report, FHL department row: <https://research.washington.edu/research/wp-content/uploads/2020/03/ARFY2019_v8.pdf>
- NIH NRSA FY2024 stipends (NOT-OD-24-104); BLS OEWS May 2025 via O*NET OnLine; UW fringe benefit rates FY2024/FY2025: <https://web.archive.org/web/20250216174147/https://finance.uw.edu/maa/files/fa/fy-2025-fringe-benefit-rates-preliminary.pdf>
- Donor share: Tide Bite, April 2023: <https://web.archive.org/web/20240118033054/https://fhl.uw.edu/about/news-and-events/2023/03/21/tide-bite-april-2023/>
- Peer studies (output and jobs ratios): `people_and_local/studies.json` (Carteret County 2018; Bigelow 2025).

**CPI:** BLS CPI-U, CUUR0000SA0 (<https://data.bls.gov/timeseries/CUUR0000SA0>).
