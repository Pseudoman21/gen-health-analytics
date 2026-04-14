# Gen-Health Analytics
### A Digital Dashboard for Visualizing Hereditary Risk and Chronic Disease Predisposition

Gen-Health Analytics is a preventive health informatics tool that bridges the gap between genealogical research and clinical risk assessment. It structures multi-generational family medical data to identify hereditary risk patterns for cardiovascular disease, type 2 diabetes, and cancers — helping you understand whether conditions in your family are driven by genetics, shared lifestyle, or both.

---

## Getting Started

### Prerequisites

- Python 3.9+
- pipenv (recommended) or pip

### Installation

```bash
pipenv install
# or
pip install -r requirements.txt
```

### Running the Dashboard

```bash
pipenv run streamlit run app.py
```

Opens at `http://localhost:8501` in your browser.

### Running the Engine via CLI

```bash
# Run with a JSON input file
python main.py my_family.json

# Run with stdin
cat my_family.json | python main.py
```

Output is JSON written to stdout. See `schemas/input_schema.json` and `schemas/output_schema.json` for the full data contracts.

---

## How to Use the Dashboard

### Step 1 — Enter your information (Proband)

In the left sidebar, fill in:
- **Your age** — used to calibrate risk thresholds and onset trend projections
- **Biological sex** — affects which cancer and CVD rules apply

### Step 2 — Select conditions to analyse

Choose one or more conditions from the multiselect dropdown. You only need to select conditions you are concerned about — the engine will only score those.

**Supported conditions:**
- `diabetes`
- `heart_disease`
- `breast_cancer`
- `ovarian_cancer`
- `male_breast_cancer`
- `colorectal_cancer`
- `hypertension`

### Step 3 — Add family members

Click **+ Add family member** for each relative you have health information about. For each member:

| Field | What to enter |
|---|---|
| **Name** | Optional — just for your reference |
| **Relationship** | e.g. mother, father, paternal_grandfather, sister |
| **Sex** | The relative's biological sex |
| **Deceased** | Check if the relative has passed away |
| **Add condition** | Select a condition from the dropdown |
| **Age onset** | The age they were diagnosed (enter 0 if unknown) |
| **+ Add** | Click this to save the condition to the member |
| **Lifestyle flags** | Checkboxes for shared risk factors (smoking, high BP, etc.) — appear automatically for relevant conditions |

> **Important:** After selecting a condition and entering the age, you must click **+ Add** to save it. The condition will then appear in the member's expander label (e.g. "Patricia — breast cancer").

You can add multiple conditions per member. Click **Remove member** to delete a member entirely.

### Step 4 — Analyse

Click **Analyse Family History**. Results appear instantly across five tabs.

---

## Dashboard Tabs Explained

### Risk Gauges

Shows two types of cards depending on the condition:

**For diabetes and heart disease** — two semicircular gauges per condition:
- **Genetic score (0–100):** How much of the elevated risk is explained by hereditary factors. Derived from the Relative Risk Ratio after subtracting the lifestyle contribution.
- **Environmental score (0–100):** How much of the risk is explained by shared lifestyle factors (smoking, obesity, high BP, etc.) among affected relatives.
- **RR (Relative Risk Ratio):** Your estimated risk compared to the general population. RR 2.0 means twice the baseline risk. RR 1.0 means no elevation above baseline.

**For cancer conditions** — a rule-based flag card:
- Shows whether any NCCN (National Comprehensive Cancer Network) clinical criteria were triggered
- Displays the number of affected relatives entered and the number of flags triggered
- Lists the specific reasons (e.g. "2+ relatives with breast cancer before age 50 — BRCA1/2 risk")
- Cancer conditions do not produce a numeric RR — the flag system is the clinical standard for hereditary cancer assessment

### Onset Trends

A line chart plotting the age of diagnosis for each condition across generations (grandparents → parents → you → children). Each data point is a relative who was diagnosed.

- A **downward slope** (trend line going lower) means the condition is appearing at a younger age each generation — a significant warning sign
- A **flat or upward slope** means onset age is stable or later
- The dotted line is an OLS regression projection — it extrapolates the trend toward your generation

Requires data from at least two different generations to draw a trend line.

### Family Tree

An interactive genogram (clinical pedigree diagram) showing all entered family members colour-coded by risk level:

| Colour | Meaning |
|---|---|
| Red | High-risk condition affecting this relative |
| Orange | Medium-risk condition |
| Teal | Affected by a condition, no elevated risk flag |
| Dark blue | You (the proband) |

- **Square** = male, **Circle** = female, **Diamond** = other
- A **†** symbol marks deceased relatives
- Lines show family structure: couple bars connect parents, drop lines connect to children

### Alerts

Prioritized clinical alerts, sorted from most urgent to least:

| Priority | What triggers it |
|---|---|
| **HIGH** | NCCN cancer red flag, RR >= 3.0, strongly accelerating onset trend, DPF >= 1.0, early-onset CVD in a relative |
| **MEDIUM** | RR >= 2.0, accelerating onset trend, DPF >= 0.7 |
| **LOW** | RR >= 1.5 (mildly elevated above baseline) |

Each alert includes a **recommended action** — a specific clinical step such as requesting genetic counselling, scheduling a screening test, or discussing with a GP.

### Raw JSON

The full engine output as JSON. Useful for debugging or exporting results. Contains all scores, DPF values, onset trend slopes, lifestyle attribution fractions, and alert details.

---

## Key Terms

| Term | Plain English |
|---|---|
| **Proband** | You — the person being assessed |
| **Relative Risk Ratio (RR)** | Your risk compared to the general population. RR 2.0 = twice the average risk |
| **DPF Proxy** | Diabetes Pedigree Function — a score derived from how many first and second-degree relatives have diabetes. Higher = more hereditary diabetes risk |
| **Genetic Predisposition Score** | 0–100 score of how much your elevated risk comes from inherited factors |
| **Environmental Risk Score** | 0–100 score of how much your elevated risk comes from shared lifestyle (diet, smoking, etc.) shared among affected relatives |
| **Lifestyle Attribution Fraction** | The proportion of family risk explained by shared lifestyle flags. 0 = fully genetic; 1 = fully lifestyle |
| **NCCN Red Flag** | A clinical criterion from the National Comprehensive Cancer Network guidelines. Triggering one means your family pattern matches a known hereditary cancer syndrome |
| **BRCA1/2** | Genes associated with hereditary breast and ovarian cancer. Triggered when 2+ first/second-degree relatives have breast cancer before age 50, or any relative has ovarian cancer |
| **Lynch Syndrome** | A hereditary condition causing colorectal cancer. Triggered when 3+ relatives across 2+ generations have colorectal cancer |
| **First-degree relative** | Parent, sibling, or child — shares ~50% of your DNA |
| **Second-degree relative** | Grandparent, aunt, uncle, grandchild — shares ~25% of your DNA |
| **Genogram** | A clinical pedigree diagram showing family structure with health information overlaid |
| **Onset Trend** | Whether a condition is appearing at a younger age in each successive generation |
| **OLS Regression** | The statistical method used to draw the trend line across generations (Ordinary Least Squares) |
| **Early-onset CVD** | Heart disease diagnosed before age 55 in a male relative or age 65 in a female relative — a significant hereditary risk marker |

---

## Conditions and How They Are Scored

### Diabetes
Uses a **DPF Proxy** (Diabetes Pedigree Function) derived from the number and closeness of relatives with diabetes. The proxy is mapped to an observed outcome rate from the BRFSS 2015 dataset (253,680 respondents) to compute the RR. First-degree relatives with diabetes contribute more weight than second-degree relatives.

### Heart Disease
Uses a **family history multiplier** derived from the EarlyMed dataset (70,000 rows). If any relative has heart disease, RR = 2.316. If that relative was also early-onset (before 55 for males, 65 for females), the RR is multiplied by a further 1.30 penalty.

### Cancer (Breast, Ovarian, Colorectal)
Uses **NCCN Clinical Practice Guidelines v2.2024** — the same rule set used by genetic counsellors in clinical practice. No population RR is calculated. Instead, the engine checks whether your family pattern matches any of the known high-risk criteria and flags accordingly.

---

## Example Scenarios

### Scenario 1 — BRCA Cancer Risk
**Setup:** Age 32, female. Add mother (breast cancer, age 44), maternal aunt (breast cancer, age 48), maternal grandmother (ovarian cancer, age 60).
**Expected:** 3 high-priority alerts — BRCA pattern from 2 breast cancer relatives under 50, plus ovarian cancer flag.

### Scenario 2 — Early-Onset CVD
**Setup:** Age 38, male. Add father (heart disease, age 48), paternal grandfather (heart disease, age 55), paternal uncle (heart disease, age 51).
**Expected:** Heart disease RR ~3.0, onset trend chart shows downward slope, high-priority CVD alert.

### Scenario 3 — Diabetes (Lifestyle vs Genetic)
**Setup:** Age 45, female. Add mother (diabetes, age 52, HighBP + HighChol flags), maternal grandmother (diabetes, age 60, HighBP), sister (diabetes, age 40, HighBP + Smoker).
**Expected:** Elevated diabetes RR, Environmental score higher than Genetic score — shared lifestyle explains most of the risk.

---

## Technical Implementation

- **Engine:** Python — validates input, runs risk scoring, produces JSON output
- **Dashboard:** Streamlit + Plotly — interactive web UI, no server required
- **Datasets (offline reference):** BRFSS 2015 diabetes survey (253,680 rows), EarlyMed heart disease dataset (70,000 rows). Constants are pre-derived — the CSVs are not loaded at runtime
- **Cancer rules:** NCCN Clinical Practice Guidelines in Oncology v2.2024

### Testing

```bash
# Install test dependencies
pipenv install --dev

# Start the app in one terminal
pipenv run streamlit run app.py

# Run Playwright UI tests in another terminal
pipenv run pytest tests/ -v

# Run with visible browser
pipenv run pytest tests/ -v --headed --slowmo 500
```
