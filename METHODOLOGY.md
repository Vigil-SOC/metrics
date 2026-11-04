# Methodology

This note defines each figure published in this repository, states where it
comes from, and records its limits. It is copied here on every run from the
private repository that collects the data, so that the definitions and the
numbers always come from the same revision.

## Source

GitHub's REST API is the only source. A scheduled job in a private operations
repository of the Vigil-SOC organization calls the following endpoints once a
day, shortly after 02:00 UTC, for `Vigil-SOC/vigil`:

| Endpoint | Feeds |
| --- | --- |
| `GET /repos/{owner}/{repo}/traffic/views` | `traffic_daily.csv`: views, unique_visitors |
| `GET /repos/{owner}/{repo}/traffic/clones` | `traffic_daily.csv`: clones, unique_cloners |
| `GET /repos/{owner}/{repo}` | `repo_daily.csv` |
| `GET /repos/{owner}/{repo}/stats/contributors` | `contributors.csv` |

The job records what GitHub returns. It applies no filter, smoothing, or
exclusion to any series.

## Retention

GitHub keeps traffic data for fourteen days and then discards it. Each run
retrieves the trailing fourteen days and writes them over the matching rows in
the archive, so a day's figure is revised until it leaves the window and is
then fixed. The archive begins on 2026-07-31, the earliest day available when
the job first ran. Repository statistics (stars, forks, watchers, open issues)
have no history endpoint; `repo_daily.csv` begins on 2026-08-14, the day that
series was added.

## Files

### traffic_daily.csv

One row per UTC day.

| Column | Definition |
| --- | --- |
| `date` | UTC calendar day |
| `views` | Page views of the repository on github.com |
| `unique_visitors` | Distinct viewers that day, as GitHub counts them |
| `clones` | Git clone operations against the repository |
| `unique_cloners` | Distinct cloners that day, as GitHub counts them |

A clone registers whenever a client runs `git clone` against the repository:
a person, a CI pipeline on every job, or a crawler. Raw clone counts therefore
overstate reach. GitHub does not say how it distinguishes a unique visitor or
cloner; the figures are published as returned.

Uniques are counted per day. A monthly or longer sum of `unique_visitors` or
`unique_cloners` counts a returning party once for each day it returns. Such a
sum is a count of visitor-days or cloner-days, not of distinct parties.

### repo_daily.csv

One row per UTC day, recorded at run time.

| Column | Definition |
| --- | --- |
| `date` | UTC calendar day |
| `stars` | `stargazers_count` |
| `forks` | `forks_count` |
| `watchers` | `subscribers_count`, the accounts subscribed to notifications |
| `open_issues` | `open_issues_count`, which GitHub defines to include open pull requests |

Stars are self-reported in practice: since mid-2026 only a repository's
administrators and collaborators can list the accounts behind a star count, so
no outsider can audit it. The series is published for completeness and should
carry little weight.

### contributors.csv

One row per contributor account, recomputed in full on every run from the
contributor statistics endpoint. Commits with no linked GitHub account are
omitted by that endpoint.

| Column | Definition |
| --- | --- |
| `login` | GitHub account |
| `account_type` | `User` or `Bot`, as GitHub reports it |
| `is_bot` | `true` when `account_type` is `Bot`, or when the login is on the agent list below |
| `commits` | Commits to the default branch attributed to the account |
| `additions` | Lines added across those commits |
| `deletions` | Lines deleted across those commits |
| `first_week` | Start of the first week with a commit |
| `last_week` | Start of the most recent week with a commit |

The agent list names accounts that GitHub reports as `User` but that commit on
behalf of coding agents: `claude`, `cursoragent`. The list is maintained in the
publisher and changes to it appear in this note. The contributor total quoted
anywhere by the project is the number of rows in this file; the bot count is
the number of rows with `is_bot` true.

### monthly_summary.csv

One row per calendar month, derived from the two daily files by summation and
month-end selection. The current month is partial; `days_recorded` says how
many days contribute.

| Column | Definition |
| --- | --- |
| `month` | `YYYY-MM` |
| `days_recorded` | Days of traffic data present for the month |
| `views` | Sum of daily views |
| `unique_visitor_days` | Sum of daily unique_visitors (see the note on uniques) |
| `clones` | Sum of daily clones |
| `unique_cloner_days` | Sum of daily unique_cloners (see the note on uniques) |
| `stars_end` | Stars on the last recorded day of the month |
| `forks_end` | Forks on the last recorded day of the month |
| `watchers_end` | Watchers on the last recorded day of the month |
| `open_issues_end` | Open issues on the last recorded day of the month |
| `stars_change` | `stars_end` minus the prior month's `stars_end`; blank for the first month |
| `forks_change` | `forks_end` minus the prior month's `forks_end`; blank for the first month |

### dashboard.html

A single HTML file with the published CSV data inlined. It loads nothing from
the network and renders offline from a clone. It is generated; do not edit it.

## What is withheld

GitHub also reports the top referring sites and the most visited paths, as
trailing fourteen-day aggregates. The job collects both and keeps them private.
Referrer data can carry the hostnames of organizations that reach the
repository from internal systems. Those organizations did not agree to be
named. The publisher refuses to run if any file outside its allowlist, or any
referrer or path value, reaches the output.

## Known distortions

- From 2026-10-19 to 2026-10-24 daily clones ran between 3,876 and 15,173
  while unique cloners stayed below 650 and views below 170. The project reads
  this as automated pipelines cloning repeatedly, not as new users. The rows are
  published as GitHub reported them.
- The `claude` account carries a large share of commits and line changes. It
  is on the agent list and is flagged `is_bot`.

## Corrections

To question a figure, propose a measure, or report a flaw in this method, open
an issue in this repository. Corrections to the data itself are made in the
private collector and appear here on the next run; the commit history of this
repository records every revision.
