---
date: 2026/10/12
jupytext:
  encoding: '# -*- coding: utf-8 -*-'
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
short_title: timeseries, irregular
---

# time series: events

```{admonition} to download
:class: warning

to work on this assignment locally, {download}`start by downloading the zip<./ARTEFACTS-timeseries-events.zip>`
```

In the previous assignment on time series, the data came **on a regular grid**: one measure each hour, one day after the other.  
Here we study a series where the timestamps are **not regular**: a catalog of **earthquakes**
in California, where each row is an *event* that happens whenever it happens.

In this assignment we practice

* recognizing that a series is **not on a fixed interval**, and measuring the gaps with `diff()`
* using `resample()` as the way to **turn events into a regular series** (counts, maxima per day, week, month)
* **zooming** on a burst of events
* **rolling** windows, that are defined by a duration rather than by a number of rows
* and, optionally, going back and forth between irregular and regular series with `reindex()`

here is the kind of picture we will end up with

<img src="media/quakes-daily.svg" width=600px>

```{code-cell} ipython3
import pandas as pd
import matplotlib.pyplot as plt

print(f"using pandas {pd.__version__}")

# time series look better in a wide figure
plt.rcParams["figure.figsize"] = (10, 4)
```

```{code-cell} ipython3
# optional
import itables
itables.init_notebook_mode()
```

## the data

+++

there is one file `data/quakes.csv`, with one row per earthquake of magnitude 2.5 or more
in California and surrounding areas, over one year (from 2025-10-01 to 2026-10-01)

| column | content |
|-|-|
| `time` | when the earthquake happened, in UTC |
| `latitude`, `longitude` | where |
| `depth` | in km |
| `mag` | the magnitude |
| `place` | a text description, like `"20 km ESE of Silver Springs, Nevada"` |
| `type` | the kind of event: nearly always `earthquake` |

+++

## 1. loading

+++

1. load the file into a dataframe `df`, and look at it; what is the type of the `time` column ?
1. turn the `time` column into timestamps, and use it as the index of `df`; make sure the index is sorted
1. the column `type` is nearly always `"earthquake"`; keep only these rows  
   (how many rows do you get rid of ? and what were they ?)

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

:::{admonition} hint
:class: tip dropdown

compare with the previous assignment: here all the strings have the same ending (`Z` stands for UTC)  
so there is no need for any option and `pd.to_datetime()` just works
:::

+++

## 2. is it regular ?

+++

in the previous assignment we have used `asfreq()` or `date_range()` to check that we had one row per hour  
here we are in a different situation: the time between 2 consecutive rows is not constant

1. compute the time that separates 2 consecutive earthquakes as a `Series` named `gaps`  
   what is the type of the values in it ? (do we get durations ?)
1. express these gaps in **minutes**, as floats; what are the shortest, the longest, and the median gaps ?
1. draw a histogram of the gaps; what do you see ?

:::{admonition} hint
:class: tip dropdown

* `diff()` on a series of timestamps (a column, or `index.to_series()`) gives the difference with the previous row
* the `.dt` accessor has a method `total_seconds()` for durations
:::

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

## 3. from events to a regular series

+++

as we have seen, `resample()` is like a `groupby` where the groups are the time intervals  
it is the tool of choice to **go from events to a regular series**

1. compute `daily`, the **number of earthquakes per day**, and plot it  
   what is the busiest day ?
1. how many days have no earthquake at all ? is this a missing data issue, like in the previous assignment ?
1. compute and plot the **maximum magnitude per day**; how many `NaN` do you get, and why ?
1. same question (number of earthquakes) per week, and per month

:::{admonition} hint
:class: tip dropdown

* to count the rows in each group, you can use `.size()`
* `"D"` is the alias for days, `"W"` for weeks and `"ME"` for months
:::

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

## 4. zooming on a burst

+++

in the daily counts, you can see a peak, that corresponds to one big earthquake followed by its aftershocks

1. what is the date of the strongest earthquake, and what is its magnitude ?
1. select the rows from 1 hour before this earthquake until 6 hours after it  
   how many are they ? and what are the magnitudes of these ?
1. compute the number of earthquakes **per hour** in that window; what do you see ?

:::{admonition} hint
:class: tip dropdown

* `idxmax()` gives the index of the maximum, so here a timestamp
* you can add or remove a duration to a timestamp, e.g. with `pd.Timedelta("1h")`
:::

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

## 5. rolling windows

+++

the daily counts are quite noisy; let us use a **rolling window** to smooth them, as we could do on a regular series  
but here a difference is that **the rows are not evenly spaced in time**, so a window of 7 rows is not a window of 7 days

1. compute `last_7d`, a dataframe with 2 columns `count` and `max`, which give, for each earthquake, the number of earthquakes and the maximum magnitude **over the last 7 days**  
   hint: `rolling()` accepts a duration, like `"7D"`, in place of a number of rows
1. plot the `count` column; what happens to it after the big earthquake ?
1. for comparison, what would be the meaning of `df.mag.rolling(7).max()` ? how long, on average, is the period that it covers ?

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

## 6. back to a grid, with `reindex()` (optional)

+++

sometimes, we need to know **the state at some given times** - for example, what was the magnitude of the latest earthquake at midnight each day ?  
for this we need to build a regular index, and to "look up" in an irregular series

1. build `midnights`, the `DatetimeIndex` with one timestamp per day at midnight (UTC), from the first to the last day of the series  
   hint: `pd.date_range()`
1. what do you get with `df.mag.asfreq("D")`, or with `df.mag.reindex(midnights)` ? how many values are defined ? why ?
1. use `reindex()` with the option `method="ffill"` to get, for each midnight, the magnitude of the latest earthquake before that time

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

## 7. the week at a glance (optional)

+++

do earthquakes occur more often at some hours, or some days ?  
since we are in California, let us look at the **local time** rather than UTC

1. convert the index to the time zone `"America/Los_Angeles"`, and add columns `hour` and `dayofweek` to `df`
1. build a dataframe with the 24 hours as rows and the 7 days of the week as columns, with the **number of earthquakes** in each cell  
   once with `pivot_table`, and once with `groupby` + `unstack`; check that you get the same
1. display that as a heatmap, using `sns.heatmap()` or `plt.imshow()`; what do you conclude ?

```{code-cell} ipython3
:tags: [level_basic]

# your code
```

:::{admonition} hint
:class: tip dropdown

to count with `pivot_table`, you can pass `aggfunc="count"`  
with `groupby`, you will use `size()`
:::

+++

## appendix: where the data comes from

+++

* the earthquakes come from the catalog of the *USGS* (US Geological Survey), that can be queried at  
  <https://earthquake.usgs.gov/fdsnws/event/1/>
* the script `fetch-data.py`, that comes with this assignment, was used to acquire this data
  - it selects a rectangle around California (so that you see some events in Nevada and Mexico too)
  - and only the events with a magnitude of 2.5 or more
* the data is frozen, no need to refresh it
