# Johnstown Castle Weather Analysis

A Jupyter notebook that turns 18 years of daily weather observations from the Johnstown Castle station in Co. Wexford, Ireland into a picture of what a normal year looks like there, and how far any single year strays from it.

The record runs from September 2008 to January 2026, 6,342 clean days. The notebook builds a day-by-day temperature climatology, measures how often it rains, finds the seasonal rainfall pattern, and compares one chosen year against the long-term profile.

## What it finds

- Rain falls on **57.9%** of days, using a 0.1 mm threshold to exclude trace drizzle
- **October** is the wettest month on average at 137 mm, **May** the driest at 61 mm, a 2.2x spread
- Half of all days have a maximum between **10.0 and 16.6 C**, a narrow band that reflects the maritime climate
- In 2020, 59.8% of days fell inside the normal range for their date, with the year averaging just 0.05 C above the long-term mean

## The three charts

**Monthly rainfall over time.** Every month in the record as one point, showing how spiky monthly totals are and whether recent years stand out.

**Average rainfall by month.** Twelve bars shaded against the median month, with the wettest highlighted, showing the seasonal shape directly.

**One year against the long-term profile.** Four layers on one axis: a shaded band for the interquartile range of each calendar day, the long-term mean, the coldest maximum ever recorded on each date, and the chosen year's actual readings. Where the black line leaves the band, that day was unusual for the time of year.

## Files

```
weather_analysis.ipynb             The notebook, with all outputs and charts saved in
Data Johnstown Castle.csv          Raw daily observations
monthly_rainfall_timeseries.png    Generated on run
avg_monthly_rainfall.png           Generated on run
temperature_comparison.png         Generated on run
```

## Notes on the data

The raw file is a Met Eireann daily export and needs handling before pandas can read it:

- **The table does not start at line 1.** Roughly 24 lines of station metadata come first. The notebook scans for the first line starting with `date,` rather than hardcoding a row number, so the same code survives a re-export with a different metadata block. The trailing comma in that search matters, because the metadata itself contains a line reading `date:  -  00 to 00 utc`.
- **Missing values are written as single spaces**, not empty fields, so `na_values=[' ', '']` is needed for pandas to see them as missing.
- **Several columns share the name `ind`**, quality indicator flags sitting beside each measurement. Pandas renames the duplicates automatically and none of them are used here.
- **Dates are lowercase**, like `19-sep-2008`, parsed with `%d-%b-%Y`.

Rows are only dropped when the date will not parse, or when all three core measurements are missing. Losing a row for a single missing field would throw away usable data.

## Method

The temperature climatology groups every observation by calendar day, ignoring the year, so all 18 first-of-Januaries collapse into one row. That gives a mean and a quartile spread for each of the 366 calendar days. The spread is the important part, since a mean line alone hides how much a given date varies year to year.

Rainfall seasonality is calculated by totalling each calendar month within each year first, then averaging those totals across years. Doing it the other way round, averaging all daily values in a month, would weight months by how many observations they happen to have rather than by their actual monthly total.

## Running it

Python 3.8 or later, with:

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

Open the notebook and run all cells, or run it headless:

```bash
jupyter nbconvert --to notebook --execute weather_analysis.ipynb
```

Keep `Data Johnstown Castle.csv` in the same folder. The notebook reads it by relative path and writes the three PNGs alongside it.

To analyse a different year, change `target_year` in section 3. If that year is not in the data, the notebook falls back to the most recent one available. To use a different station, any Met Eireann daily export with `date`, `maxtp`, `mintp` and `rain` columns will work without changes, since the header detection handles a different metadata block on its own.
