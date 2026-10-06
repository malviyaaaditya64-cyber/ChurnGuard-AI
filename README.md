# ChurnGuard

**Client-side customer churn analytics — upload a spreadsheet, get a trained risk model, no backend required.**

ChurnGuard is a single-file web app that takes a CSV or Excel export of your customer data, automatically figures out what each column means, and scores every customer's churn risk. If your data includes a churn/cancellation column, it trains a real logistic regression model in the browser and reports a holdout accuracy. If it doesn't, it falls back to a transparent rule-based score and says so — it never pretends a heuristic is a trained model.

---

## What it does

- **Upload a CSV or Excel (.xlsx/.xls) file** — one row per customer, any columns you track.
- **Auto-detects column roles** by name and content: customer ID, name, churn label, revenue/value, tenure, engagement, support tickets.
- **Trains a logistic regression model in-browser** (gradient descent, L2 regularization, 80/20 train/holdout split) when a usable churn column is found — with an honestly reported holdout accuracy, not a made-up number.
- **Falls back to a rule-based score** when there's no churn label, and clearly labels itself as "rule-based" in the UI rather than overclaiming.
- **Explains every score**: each customer's risk card shows the top factors pushing their risk up or down, computed directly from the model's coefficients — not templated text.
- **What-if simulator**: drag a slider to change a customer's top risk driver and watch the predicted risk recalculate live, using the same trained model.
- **Dashboards**: revenue-at-risk, overall risk gauge, risk distribution, churn drivers chart, and risk-by-tenure-cohort chart.
- **Searchable, sortable, filterable customer ledger.**
- **Export** the fully scored dataset back out as CSV, or print a clean report view.
- **Sample dataset generator** built in, so you can try it with zero setup.

## Tech stack

Everything runs client-side in a single `index.html` — no server, no build step, no backend.

| Purpose | Library |
|---|---|
| CSV parsing | [PapaParse](https://www.papaparse.com/) |
| Excel parsing | [SheetJS (xlsx)](https://sheetjs.com/) |
| Charts | [Chart.js](https://www.chartjs.org/) |
| Modeling | Hand-written logistic regression (vanilla JS, no ML library) |
| Fonts | Fraunces, IBM Plex Sans, IBM Plex Mono (Google Fonts) |

All three libraries load from `cdnjs.cloudflare.com` and Google Fonts, so **an internet connection is required when the page is opened** — but no installation or build process is needed.

## Getting started

Clone the repo and serve the folder with any static file server — **do not open `index.html` directly via `file://`**, since the browser blocks some script behavior on the local file protocol.

```bash
git clone https://github.com/<your-username>/ChurnGuard-AI.git
cd ChurnGuard-AI
python -m http.server 8000
# then open http://localhost:8000
```

Or use VS Code's **Live Server** extension and click "Open with Live Server" on `index.html`.

### GitHub Pages

1. Push `index.html` to the repo (root of `main` branch is fine).
2. Go to **Settings → Pages**.
3. Set source to `main` branch, `/ (root)`.
4. Your live demo will be at `https://<your-username>.github.io/ChurnGuard-AI/`.

## Expected data format

Any CSV or Excel file with a header row works. Columns are matched by name, so exact headers don't matter much — these are recognized automatically:

| Role | Matches headers containing |
|---|---|
| Customer ID | `id`, `customer id`, `account id` |
| Customer name | `name`, `customer name`, `email` |
| Churn label | `churn`, `exited`, `attrition`, `cancelled`, `status` (binary values like Yes/No, 1/0, Active/Churned) |
| Revenue / value | `revenue`, `mrr`, `ltv`, `value`, `amount`, `charge`, `price` |
| Tenure | `tenure`, `months since`, `duration`, `account age` |
| Engagement | `engagement`, `usage`, `activity`, `login`, `nps`, `satisfaction` |
| Support tickets | `ticket`, `complaint`, `support`, `issue` |

If no churn column is found, ChurnGuard still runs — it scores accounts using a transparent, name-based heuristic (e.g., low engagement + low tenure + high ticket volume → higher risk) and labels the result "rule-based" rather than implying it's a trained model.

A ready-to-use sample file (`sample_customers.csv`) is included for testing.

## How the model works (short version)

1. Numeric columns are standardized (z-score). Low-cardinality categorical columns are one-hot encoded.
2. If a churn column exists with enough examples of both classes, the data is split 80/20 and a logistic regression is trained with batch gradient descent and L2 regularization.
3. Holdout accuracy is computed on the untouched 20% and shown in the UI, so the reported performance isn't just measured on data the model already saw.
4. The final model is refit on the full dataset for scoring.
5. Each customer's risk explanation is the model's coefficients multiplied by that customer's standardized feature values — the same numbers driving the prediction, not a separate narrative layer.

## Limitations

- This is a from-scratch, dependency-free logistic regression meant for clarity and a reasonably-sized dataset in the browser — it is not a production ML pipeline and won't scale to very large files.
- There's no persistence or backend: nothing is saved between sessions, and no data ever leaves the browser.
- The rule-based fallback is a heuristic, not a model — treat its output as directional, not predictive.

## License

MIT — use it, modify it, ship it.

---

Built as a business analytics / AI portfolio project.
