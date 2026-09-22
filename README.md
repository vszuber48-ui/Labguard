# LabGuard

A client-side data-quality tool for messy lab and clinical CSV exports. It runs
standard cleaning (duplicate rows, inconsistent casing/whitespace, mixed date
formats, missing values) and then checks every recognized measurement against
a **physiologically plausible range**, not just a "normal" reference range —
so it flags likely transcription or unit errors without misreading a genuinely
abnormal (but real) result as bad data.

**[Live demo →](#)** *(replace with your GitHub Pages URL once deployed — see below)*

## Why plausibility, not "normal"

A sick patient can have a real glucose reading of 300 mg/dL — that's abnormal,
not wrong. But a diastolic blood pressure of 0, or a hemoglobin of 45 g/dL, is
not a finding, it's a data-entry error: no living person's body produces that
number. LabGuard only flags values outside what's physiologically possible,
which is a much stronger and more defensible signal than "this looks unusual."

## What it checks

| Field | Plausible range | Common false-value LabGuard catches |
|---|---|---|
| Glucose | 20–600 mg/dL | Value in mmol/L range (2.5–8.5) left unconverted |
| Hemoglobin | 2–24 g/dL | Decimal-place / unit slip |
| Systolic BP | 40–300 mmHg | Missing or extra digit |
| Diastolic BP | 20–200 mmHg | Missing or extra digit |
| Heart rate | 20–250 bpm | Transcription error |
| Temperature | 30–43 °C | Fahrenheit value (≈90–108) stored in a Celsius field |
| BMI | 8–80 kg/m² | Height/weight unit mismatch |
| WBC count | 200–50,000 /µL | Magnitude/decimal slip |
| Platelets | 5,000–1,500,000 /µL | Missing or extra zero |
| Age | 0–122 years | Sign error, birth-date subtraction bug |

Column names are matched flexibly (e.g. `hemoglobin_g_dl`, `Hb`, and
`haemoglobin` all resolve to the same check), and generic cleaning — exact
duplicate rows, mixed date formats, blank cells, inconsistent sex/gender
casing — runs regardless of which clinical fields are present.

## Validated against real public data

`sample-data/` includes:

- `synthetic_messy_sample.csv` — a small, hand-built example with deliberately
  planted errors (used by the in-app "Load sample data" button).
- `real_data_pima_diabetes.csv` — the real Pima Indians Diabetes dataset
  (National Institute of Diabetes and Digestive and Kidney Diseases, via the
  [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/34/diabetes)).
  Only a header row with the documented real column names was added; no values
  were changed.
- `real_data_heart_disease.csv` — the real UCI/Cleveland Clinic Heart Disease
  dataset ([UCI ML Repository](https://archive.ics.uci.edu/dataset/45/heart+disease)).
  Columns were renamed to their documented real meanings (`trestbps` →
  `systolic_bp`, `thalach` → `heart_rate_max_bpm`) and `sex` was decoded from
  its 1/0 code to Male/Female per the dataset's own documentation; no values
  were changed.

Running the Pima file through LabGuard flags 51 rows — every one traces back
to a well-documented real flaw in that dataset: missing values for glucose,
blood pressure, and BMI were encoded as `0` instead of left blank, which is
physiologically impossible and exactly what the plausibility check is designed
to catch. This wasn't tuned for that dataset in advance; the same rules that
catch errors in the synthetic sample caught this independently-documented
issue in real research data.

## How it works

- **Parsing**: a small hand-written CSV parser (handles quoted fields with
  embedded commas/newlines) — no external dependency.
- **Column matching**: normalizes headers and matches against known aliases
  for each clinical field, first by exact match, then by substring match for
  longer, unambiguous names (so `hemoglobin_g_dl` still resolves to
  hemoglobin without short codes like `hb` causing false positives elsewhere).
- **Checks**: each matched field is tested against its plausible range; sex/
  gender values are normalized to M/F; dates are parsed loosely and
  standardized to ISO (`YYYY-MM-DD`) where the format is unambiguous.
- **Everything runs in the browser.** No data is uploaded anywhere — parsing,
  checks, and the "copy cleaned CSV" output all happen client-side.

## Tech stack

Plain HTML, CSS, and JavaScript. No build step, no framework, no backend.

## Running it locally / hosting on GitHub Pages

1. Push this repo's contents to a new GitHub repository (`index.html` must be
   at the repo root, or in `/docs` if you configure Pages that way).
2. In the repo's **Settings → Pages**, set the source to your default branch.
3. GitHub will publish it at `https://<your-username>.github.io/<repo-name>/`.

No dependencies to install — it's a single static file.

## Scope and limitations

LabGuard is a data-quality demo, not a diagnostic or clinical tool. It flags
values that are unlikely to be real given human physiology; it does not
interpret health status, and nothing it outputs is medical advice. Reference
ranges used are standard, widely published plausibility bounds for QA
purposes, not clinical diagnostic thresholds.
