# Vigil adoption figures

Daily adoption figures for [Vigil-SOC/vigil](https://github.com/Vigil-SOC/vigil),
published unaltered from GitHub's API so that anyone can check what the project
claims. A scheduled job in a private operations repository collects the data
each day and commits an allowlisted subset here. This repository holds data and
documentation only: no workflows, no scripts, no credentials.

Why this exists: the post *Numbers You Can Check* on [vigilsoc.org](https://www.vigilsoc.org).

## Contents

| Path | Holds |
| --- | --- |
| `data/traffic_daily.csv` | Views, unique visitors, clones, unique cloners, one row per UTC day since 2026-07-31 |
| `data/repo_daily.csv` | Stars, forks, watchers, open issues, one row per day since 2026-08-14 |
| `data/contributors.csv` | Commit history per contributor account, with a bot flag |
| `data/monthly_summary.csv` | Monthly totals and month-end values derived from the two daily files |
| `METHODOLOGY.md` | What each column means, where it comes from, and its limits |
| `dashboard.html` | A self-contained chart of the files above; works offline |

The commit history is the audit trail. Each daily commit revises the trailing
fourteen days of traffic as GitHub revises them, and the methodology note is
copied in with the data so the two never drift apart.

## Reading the figures

Read `METHODOLOGY.md` first. Two cautions matter most:

- Unique visitors and unique cloners are counted per day. A monthly sum counts
  a returning party once for each day it returns, so the summary columns are
  named `unique_visitor_days` and `unique_cloner_days`.
- Raw clone counts include automated pipelines. Some days show thousands of
  clones from a few hundred unique cloners. The rows are published as GitHub
  reported them, with no filtering.

## Using the data

Clone the repository and open `dashboard.html` in a browser, or load the CSV
files into any tool:

```sh
git clone https://github.com/Vigil-SOC/metrics
open metrics/dashboard.html
```

```python
import pandas as pd
daily = pd.read_csv("data/traffic_daily.csv", parse_dates=["date"])
daily.set_index("date")[["clones", "unique_cloners"]].plot()
```

## What is withheld

GitHub also reports referring sites and most visited paths. Those are collected
and kept private, because referrer data can carry the hostnames of
organizations that reach the repository from internal systems. The publisher
refuses to run if any such value reaches this repository.

## Corrections and proposals

Open an issue to question a figure, propose a measure, or point out a flaw in
the method. The data is regenerated daily from the private collector, so
corrections are made there and appear here on the next run; pull requests
against `data/` will be overwritten.

## License

The contents of this repository are released under CC0 1.0 (see `LICENSE`).
