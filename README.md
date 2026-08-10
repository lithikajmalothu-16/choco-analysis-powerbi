# Awesome Chocolates — Sales & Profitability Dashboard

A Power BI report I built to put two Microsoft certifications into practice.

I earned [Get data in Power BI] and [Get started building with Power BI] through Microsoft Learning, then wanted to find out whether I could actually build something with it rather than just pass an assessment. This is that build — a full report from raw Excel through data modeling, DAX, and layout, following [Chandoo.org's comprehensive 3 hour video course](https://youtu.be/9sEJclxSxFQ?si=SYfrYsPdjGpuqc_C) as a guide.

Sample dataset, Jan 2023 – Mar 2025. The dataset and report brief are from the course. The model, every measure, the theme, and the layout are mine.

![Dashboard](dashboard.png)

---

## What it does

A single-page executive report on a chocolate distribution business — five KPIs, year-over-year trends on revenue and volume, geographic split, order-size distribution, and top-6 rankings for products and salespeople.

Everything is cross-filtered. Clicking a product filters the salespeople; clicking a country reshapes the shipment histogram. The two CY-vs-PY trend charts are deliberately locked to a fixed 13-month window so the comparison stays stable while the rest of the page moves.

---

## Data model

Five tables in a star schema — `shipments` as the fact table (~25,000 rows, one row per shipment), with `calendar`, `locations`, `products`, and `people` as dimensions.

```
                  calendar
                     │
   locations ──── shipments ──── products
                     │
                   people
```

`calendar` is marked as a date table, which is what makes the time-intelligence measures work correctly.

---

## The measures that matter

**Profit required a Power Query step first.** `shipments` has revenue but no cost; `products` has cost per box. I merged the two on `PID`, multiplied `Boxes × Cost_per_box`, and dropped everything else — so cost exists at shipment grain before the model ever loads.

```dax
Total Profit = [Total Amount] - [Total Cost]

Profit % = DIVIDE([Total Profit], [Total Amount])
```

`DIVIDE` over `/` because slicing down to a salesperson-country combination with no shipments produces a zero denominator. `DIVIDE` returns blank; `/` throws.

**Year-over-year comparison:**

```dax
PY Amount =
CALCULATE(
    [Total Amount],
    SAMEPERIODLASTYEAR(calendar[cal_date])
)
```

**Variance, with the blank trap handled:**

```dax
Amount Variance =
VAR TA_LY = [PY Amount]
RETURN
    IF(ISBLANK(TA_LY), BLANK(), [Total Amount] - TA_LY)
```

Without the `ISBLANK` guard, the first twelve months show a variance equal to the full current-year figure — Power BI treats the missing prior year as zero. It reads as a total collapse when the real answer is "no data yet."

---

## Concepts applied

Power Query merges and derived columns · star schema modeling and filter propagation · `CALCULATE` and filter context · time intelligence · `COUNTROWS`, `DIVIDE`, `VAR`/`RETURN` · sort-by-column · binning for histograms · Top N filters · filter pane vs. slicers · Edit interactions · conditional formatting with custom bar scales · Image URL data category · custom theme JSON

---

## What I'd fix next

- Add a bottom-6 view — underperformers are usually the more actionable list
- Add a "top 6 as % of total" measure so the rankings show concentration, not just order
- Subtitles on each visual naming the dimension being sliced

---

## Files

| File | What it is |
|---|---|
| `Choco analysis.pbix` | The report (open in Power BI Desktop) |
| `bibg.json` | Custom theme — apply via View → Themes → Browse |
| `sample-chocolate-shipments-data.xlsx` | Source data |
| `dashboard.png` | Screenshot |

---

Built by [Lithika Jadav](https://github.com/lithikajmalothu-16) · Dataset and course from [Chandoo.org](https://youtu.be/9sEJclxSxFQ?si=SYfrYsPdjGpuqc_C)
