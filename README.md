# Verve Pipeline AI Tool
**Prepared by Arnav Patel | Last updated: April 2, 2026**

This tool automates the gathering of contract intelligence from OrangeSlices AI articles.
Given any OrangeSlices expiring-tasks URL, it extracts every contract listed and enriches
each one with incumbent research, spending data, SOW notes, related news, and capture
actions — then writes everything to a dated, agency-tagged Excel file.

---

## What's in this folder

```
verve_pipeline/
├── pipeline.py                  ← Main script (run this for new lists)
├── output/
│   └── VA_2026-04-02_initial.xlsx  ← Initial VA deliverable (all 24 contracts, pre-enriched)
└── README.md                    ← This file
```

---

## Setup (one time)

### 1. Install Python dependencies
```bash
pip install anthropic openpyxl requests beautifulsoup4
```

### 2. Get an Anthropic API key
- Go to https://console.anthropic.com
- Create an API key
- Set it as an environment variable:

```bash
# Mac / Linux
export ANTHROPIC_API_KEY="sk-ant-..."

# Windows (Command Prompt)
set ANTHROPIC_API_KEY=sk-ant-...
```

---

## How to run it on a new OrangeSlices list

```bash
python pipeline.py --url "https://orangeslices.ai/three-dozen-dha-expiring-tasks-expected-to-draw-a-crowd-in-2027/" --agency DHA
```

Replace the URL and agency name as needed. The script will:
1. Fetch the article
2. Extract all contracts (title, ID, value, bidders)
3. Enrich each contract using Claude AI + web search (~2-3 min total)
4. Write an Excel file to `output/DHA_2026-04-02.xlsx` (date is automatic)

**To run on the VA list:**
```bash
python pipeline.py --url "https://orangeslices.ai/three-dozen-va-expiring-tasks-expected-to-draw-a-crowd-in-2027/" --agency VA
```

---

## Excel output structure

Each run produces one `.xlsx` file with two sheets:

### Sheet 1: `{AGENCY}_{DATE}` — one row per contract
| Column | Description |
|---|---|
| Contract Title | Full name from OrangeSlices |
| Contract ID | Task order / PIID number |
| Value ($M) | Total ceiling value |
| Bidders | Number of bidders at last award |
| Agency | Agency tag (e.g. VA, DHA) |
| Date Retrieved | Auto-set to today's date |
| Incumbent | Current awardee (AI-researched) |
| Incumbent Notes | Past performance context |
| Contract Type | Task Order, IDIQ, GSA Schedule, etc. |
| NAICS Code | Primary NAICS |
| NAICS Description | NAICS description |
| Set-Aside | Full and Open, 8(a), SDVOSB, etc. |
| USASpending Note | What to look for on USASpending.gov |
| SAM.gov Note | Where to find the solicitation/SOW |
| Recompete Risk | High / Medium-High / Medium / Low-Medium / Low / Very Low |
| Recompete Rationale | 1-sentence explanation |
| Related News | Press releases, budget exhibits, news |
| Capture Actions | Top 2-3 BD actions |
| HigherGov Link | Direct link to contract on HigherGov |
| USASpending Link | Direct search link on USASpending.gov |

Recompete Risk column is color-coded: red = High, orange = Medium-High, yellow = Medium,
light green = Low/Very Low.

### Sheet 2: `Summary` — key metrics
Auto-calculated totals: contract count, total value, avg bidders, risk breakdown.

---

## Separating lists by date and agency

Each run creates a separate file named `{AGENCY}_{DATE}.xlsx`. To compare across runs:
- Open both files in Excel
- Copy each agency-date sheet into a master workbook
- Use pivot tables or filters on the Agency and Date Retrieved columns

Recommended master workbook structure:
```
Verve_Pipeline_Master.xlsx
├── VA_2026-03-27     ← First VA pull
├── VA_2026-04-02     ← Second VA pull (new list)
├── DHA_2026-04-02    ← DHA pull
└── Summary           ← Cross-agency pivot table
```

---

## Notes on the DHA article

Jennifer's first email asked about DHA contracts. The DHA 2027 article exists at:
https://orangeslices.ai/three-dozen-dha-expiring-tasks-expected-to-draw-a-crowd-in-2027/

However, the DHA contract list is behind the OrangeSlices Insider paywall (~$295/year).
If Verve has or gets a subscription, run:
```bash
python pipeline.py --url "https://orangeslices.ai/three-dozen-dha-expiring-tasks-expected-to-draw-a-crowd-in-2027/" --agency DHA
```
The script will extract whatever is publicly visible. Paywalled contracts will need to be
manually entered into the Excel using the same column structure.

---

## Initial VA deliverable

`output/VA_2026-04-02_initial.xlsx` contains all 24 publicly listed VA contracts from the
March 27, 2026 OrangeSlices report, pre-enriched with incumbent research, NAICS codes,
set-asides, spending notes, SAM.gov guidance, recompete risk ratings, and capture actions.

To regenerate this file without running the full API pipeline:
```bash
python generate_initial_output.py
```

---

## Troubleshooting

**"ModuleNotFoundError: No module named 'anthropic'"**
→ Run `pip install anthropic` first.

**"ANTHROPIC_API_KEY not set"**
→ Set the environment variable (see Setup above). The key starts with `sk-ant-`.

**Script extracts 0 contracts**
→ The article may be fully paywalled. Check the URL in a browser. If the contract list
  is not visible without logging in, the script cannot extract it automatically.

**Excel file looks blank**
→ Check that the `output/` folder exists (the script creates it automatically).
  If running `generate_initial_output.py`, ensure openpyxl is installed.
