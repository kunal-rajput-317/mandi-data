# 🌾 Mandi Bhav Data — Daily Indian Agricultural Market Prices

A clean, daily-updated, open archive of **wholesale mandi (APMC) prices** for
agricultural commodities across India — one CSV file per day, forever.

Maintained automatically by [KrashiMitra](https://krashimitra.in). Every night a
scheduler exports the previous day's prices from the live database into this
repository, so the history keeps growing even though the app itself only keeps a
short hot window.

---

## Why this exists

Agmarknet / data.gov.in publishes daily mandi prices, but the feed is **wiped and
refilled every day** — there is no easy, clean, continuous historical record.
This repo is that record: normalized, deduplicated, one immutable file per day,
ready for analysis and model training.

---

## What's inside

```
data/
  2026/
    07/
      2026-07-16.csv
      2026-07-17.csv
      ...
```

- **One file per calendar day**, named by the mandi **arrival date** (`YYYY-MM-DD`).
- Files are **immutable** once written — a given day is never rewritten.
- Plain UTF-8 CSV with a header row (renders as a browsable table on GitHub).

## Schema

Every file has the same columns, in this exact order:

| Column        | Type    | Description                                              |
|---------------|---------|----------------------------------------------------------|
| `date`        | ISO date | Arrival date the price was reported (`YYYY-MM-DD`).      |
| `state`       | string  | State / UT name, as reported by Agmarknet.               |
| `district`    | string  | District name.                                           |
| `market`      | string  | Mandi (market) name.                                     |
| `commodity`   | string  | Commodity / crop name (e.g. Wheat, Onion, Soyabean).     |
| `variety`     | string  | Variety, where reported (may be empty).                  |
| `grade`       | string  | Grade, where reported (e.g. FAQ; may be empty).          |
| `min_price`   | integer | Minimum price for the day, **₹ per quintal**.            |
| `max_price`   | integer | Maximum price for the day, **₹ per quintal**.            |
| `modal_price` | integer | Modal (most common) price, **₹ per quintal**. Use this.  |

Notes:
- All prices are **₹ per quintal (100 kg)** — the Agmarknet convention.
- Empty cells mean the source did not report that field. Missing ≠ zero.
- `modal_price` is the value most analyses want (the representative price).

---

## Quick start

```python
import pandas as pd

df = pd.read_csv("data/2026/07/2026-07-16.csv")

# Today's wheat prices, cheapest mandi first
wheat = df[df["commodity"] == "Wheat"].sort_values("modal_price")
print(wheat[["state", "market", "modal_price"]].head())
```

Load a whole month:

```python
import glob, pandas as pd

frames = [pd.read_csv(f) for f in glob.glob("data/2026/07/*.csv")]
month = pd.concat(frames, ignore_index=True)
```

---

## Update schedule

- **Automatic, daily at ~04:00 IST.** At that hour the previous day's final
  prices are complete, so each file is a settled end-of-day record.
- The job is **self-healing**: a missed run is caught up automatically, and
  re-runs never create duplicates.

## Coverage

- **Source:** [Agmarknet via data.gov.in](https://www.data.gov.in/catalog/current-daily-price-various-commodities-various-markets-mandi)
  (Variety-wise Daily Market Prices, resource `9ef84268-d588-465a-a308-a864a43d0070`).
- **Geography:** all states / UTs that report on a given day.
- **History:** grows daily from the archive start date. Older data may be
  backfilled over time.

---

## Data quality & disclaimer

This data is provided **as-is** for research and informational use. It mirrors
what the government feed reports, which can contain gaps, late corrections, or
occasional outliers. **Do not treat it as financial advice** or as an official
price of record — for transactions, verify against
[Agmarknet](https://agmarknet.gov.in) or [eNAM](https://enam.gov.in).

## License

Released under **[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)** —
free to use, share, and build on, with attribution. Underlying price data
originates from the Government of India Open Government Data (OGD) Platform.

**Attribution:** "Mandi price data via [KrashiMitra](https://krashimitra.in),
sourced from Agmarknet / data.gov.in."

---

<sub>Maintained automatically by KrashiMitra. Prices in ₹/quintal. Not affiliated
with Agmarknet or data.gov.in.</sub>
