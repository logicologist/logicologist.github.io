# Using Chrome UX Report Popularity Data

At IMC 2022, my co-authors and I showed that the Chrome User Experience Report (CrUX) is the most accurate alternative to the now-discontinued Alexa Top Million list of popular websites:

[Toppling Top Lists: Evaluating the Accuracy of Popular Website Lists](https://kcruth.com/papers/2022-Toplists.pdf). Kimberly Ruth, Deepak Kumar, Brandon Wang, Luke Valenta, and Zakir Durumeric. IMC 2022.

I've received multiple questions about how to query this dataset in practice. This document is meant to serve as a quick guide for researchers looking to use the data.

## Preliminaries

The Chrome UX Report is available on BigQuery. (As of November 2022, BigQuery has a 1TB/mo free tier; the queries in this guide fit comfortably within that limit. Source: [this page](https://cloud.google.com/bigquery/pricing).)

CrUX is publicly accessible on BigQuery via this dataset name: `chrome-ux-report`

CrUX documentation is available here: [https://developer.chrome.com/docs/crux/](https://developer.chrome.com/docs/crux/)

Website popularity is aggregated at the level of **origin**. That means subdomains are broken out separately from their parent domain, and http(s) protocol is specified as well.

Website rank is presented as **rank magnitude**: for instance, the top 1000 websites are treated as a set. The order in which they're listed doesn't matter. Starting in October 2022, CrUX buckets site ranks by half-magnitudes: 1K, 5K, 10K, 50K, 100K, etc. (Whole-step rank magnitudes are available dating back to February 2021.)

Popularity data is available both for a globally aggregated list and per country. We'll cover each below.

## Querying the global popularity list

If you're just looking for a single global popularity list, this query suffices:

```
SELECT
  origin,
  experimental.popularity.rank AS rank_magnitude
FROM
  `chrome-ux-report.all.202211`
GROUP BY origin, rank_magnitude
ORDER BY rank_magnitude
```

For just the top million, add the condition `WHERE experimental.popularity.rank <= 1000000`.

## Querying per-country breakdowns

A global list doesn't capture the full diversity of web browsing between countries. (For a deep-dive, see the other IMC paper I co-authored: [A World Wide View of Browsing the World Wide Web](https://kcruth.com/papers/2022-Browsing.pdf). Kimberly Ruth, Aurore Fass, Jonathan J. Azose, Mark Pearson, Emma Thomas, Caitlin Sadowski, and Zakir Durumeric. IMC 2022.)

Here's how I include per-country breakdowns when I query CrUX:

```
SELECT
  origin, rank_magn, country_code
FROM (
  SELECT DISTINCT
    origin, experimental.popularity.rank AS rank_magn, country_code
  FROM
    `chrome-ux-report.experimental.country`
  WHERE
    yyyymm IN (202211)
    AND
    experimental.popularity.rank <= 1000000)
FULL JOIN (
  SELECT DISTINCT
    origin, experimental.popularity.rank AS rank_magn, 'GLOBAL' AS country_code
  FROM
    `chrome-ux-report.experimental.global`
  WHERE
    yyyymm IN (202211)
    AND
    experimental.popularity.rank <= 1000000)
USING
  (origin, rank_magn, country_code)
```

Country codes are given as ISO alpha-2. I include the global list in this query also for convenience.



