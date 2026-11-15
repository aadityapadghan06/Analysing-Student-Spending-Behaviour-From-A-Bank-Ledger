# ANALYSING-STUDENT-SPENDING-BEHAVIOUR-FROM-A-BANK-LEDGER
# Student Spending Behaviour Analysis

A data analysis project that cleans a student's bank ledger, sorts every payment into needs, wants and savings, finds the unhealthy spending patterns, and proposes a quantified plan for spending more efficiently.

The analysis is written in Python and runs end to end in Google Colab or locally. It uses a **partially synthetic** ledger of 192 entries covering 14 September to 31 October 2026 (48 days).

## Contents

- [Project goals](#project-goals)
- [Key findings](#key-findings)
- [Repository structure](#repository-structure)
- [Getting started](#getting-started)
- [How the analysis works](#how-the-analysis-works)
- [Customising the analysis](#customising-the-analysis)
- [Data](#data)
- [Limitations](#limitations)
- [Tools and references](#tools-and-references)
- [License](#license)

## Project goals

1. Clean the ledger and check that its running balance is internally consistent.
2. Categorise every transaction and separate necessary spending (needs) from unnecessary spending (wants), with savings treated on its own.
3. Analyse the two most important categories, eating out and transport, in detail.
4. Analyse the sources of funding, including how much each individual provided.
5. Identify unhealthy spending patterns and propose a realistic reduction plan.

## Key findings

| Measure | Result |
|---|---|
| Total spent (149 debits) | ₹28,173 (about ₹17,600 a month) |
| Needs / Wants / Savings | 41.4% / 45.4% / 3.5% |
| Unidentified payments and friend transfers | 7.5% and 2.1% |
| Largest category | Eating out, ₹9,584 (34.0%) |
| Eating out driven by | 11 orders of ₹300 or more, which make up 61.6% of the category |
| Transport | ₹5,878; cost per day fell 60% from September to October as cab use gave way to the student's own vehicle |
| Total funding | ₹14,085 from four people (Padghan family 55.1%, Zagdu Singh 44.9%) |
| Inflows without the ₹10,407 fee refund | ₹18,575, about two thirds of spending |
| Proposed reduction | About ₹3,560 a month, bringing wants down to roughly 31.6% of spending |
| Label consistency (5-fold cross-validation) | 82% agreement between the rules and a classifier |

Funding by person:

| Funder | Transfers | Total | Share |
|---|---|---|---|
| Zagdu Singh | 1 | ₹6,325 | 44.9% |
| Atharva Padghan | 6 | ₹5,720 | 40.6% |
| Rajesh Padghan | 2 | ₹1,040 | 7.4% |
| Ashwini Padghan | 1 | ₹1,000 | 7.1% |

The full discussion, tables and figures are in the project report (see below).

## Repository structure

```
.
├── README.md
├── requirements.txt
├── data/
│   └── Financial_Ledger__September__October.xlsx      synthetic ledger
├── src/
│   ├── ledger_analysis.py                             full analysis, all phases
│   └── funding_analysis.py                            funding analysis only
├── outputs/                                           created when you run the scripts
│   ├── ledger_clean_categorised.csv
│   ├── ledger_analysis_tables.xlsx
│   ├── ledger_overview.png
│   ├── confusion_matrix.png
│   └── funding.png
└── report/
    └── Spending_Behaviour_Academic_Report.docx        17-page academic report
```

## Getting started

### Requirements

Python 3 and the packages in `requirements.txt` (pandas, numpy, matplotlib, scikit-learn, openpyxl). The scripts were tested with Python 3.13, pandas 3.0, numpy 2.5, matplotlib 3.11, scikit-learn 1.9 and openpyxl 3.1. Older versions of these packages should work but have not been tested.

```bash
pip install -r requirements.txt
```

### Run locally

```bash
python src/ledger_analysis.py data/Financial_Ledger__September__October.xlsx
python src/funding_analysis.py data/Financial_Ledger__September__October.xlsx
```

Both scripts print their tables to the terminal and write their files to the current folder. Run them from `outputs/` if you want the results kept there.

### Run in Google Colab

1. Open [colab.research.google.com](https://colab.research.google.com) and create a new notebook.
2. Upload the ledger from the Files panel on the left.
3. Paste the contents of `src/ledger_analysis.py` into a cell.
4. Set `FILE` near the top to the uploaded file's path, for example `/content/Financial_Ledger__September__October.xlsx`, and run the cell.

Colab already includes every required package.

## How the analysis works

The work follows seven phases, each with its own section in `ledger_analysis.py`.

| Phase | What it does |
|---|---|
| 1. Clean and validate | Standardises mixed date formats, trims text, fills blanks, builds a signed amount, and checks each balance against the previous balance plus the entry |
| 2. Categorise | Assigns each debit to one of twelve categories using an ordered table of keyword patterns; category totals are checked against the ledger total |
| 3. Needs and wants | Labels each debit as need, want, savings, transfer or unclear; eating out and cab rides are split by payment size |
| 4. Eating out and transport | Venue, order size, weekday and month for eating out; mode and type for transport; months compared per day because September starts on the 14th |
| 5. Funding | Tags each credit as funding, fee refund, friend reimbursement or bank credit; funding summed by person with a timeline |
| 6. Reduction plan | Applies a percentage cut to each want category and reports the saving per month |
| 7. Label check | Trains a logistic regression on the description, amount and weekday, and evaluates it with stratified 5-fold cross-validation to produce a confusion matrix |

## Customising the analysis

The parts most worth changing are near the top of their sections:

- **`RULES`** is the category table. Add or edit a pattern to move payments between categories.
- **`EAT_NEED_BELOW`** (default 100) and **`CAB_NEED_BELOW`** (default 200) are the payment sizes below which eating out and cab rides count as needs.
- **`CUTS`** holds the reduction percentages for each want category.
- **`FUNDERS`** and the credit rules decide who counts as a funder.

After any change, rerun the script and every table, chart and output file updates.

## Data

`data/Financial_Ledger__September__October.xlsx` is a synthetic ledger. It has five columns: Date, Particulars, Debit, Credit and Balance. No real person is described, but the analysis treats it as genuine.

Interpretation notes that the code relies on:

- Credits from the Padghan family and from Zagdu Singh are funding.
- A ₹10,407 credit labelled "College" is a fee refund from Zagdu Singh. It is reported separately and is not counted as income.
- Swarg is a gift shop, Rumit is a friend, and a payment to Nileshbhai Mali repaid a debt.

To use your own ledger, keep the same five columns and update the name rules in the scripts.

## Limitations

- Forty-eight days is too short to show long-run or seasonal habits, and September covers only 17 days.
- The ledger is synthetic, so the findings describe this dataset and do not generalise to other students.
- The needs and wants split depends on the thresholds above, so the wants share is an estimate.
- Fifteen payments (₹2,105) go to individuals whose purpose is unknown and sit outside the needs and wants split.
- Six balances do not reconcile with the entries around them (net about ₹65). Amounts were left unchanged.
- The classifier is trained on labels produced by the rules, so its 82% agreement shows how consistent the rules are, not whether they match the student's own judgement.

## Tools and references

- pandas: McKinney, W. (2010). Data structures for statistical computing in Python. *Proceedings of the 9th Python in Science Conference*, 51–56.
- matplotlib: Hunter, J. D. (2007). Matplotlib: A 2D graphics environment. *Computing in Science & Engineering*, 9(3), 90–95.
- scikit-learn: Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.
- The needs, wants and savings framing follows the 50/30/20 rule of thumb in Warren, E., & Tyagi, A. W. (2005). *All your worth: The ultimate lifetime money plan*. Free Press. It is applied here to shares of spending, not income, so treat it as indicative.

## Author

Aaditya Rajesh Padghan
