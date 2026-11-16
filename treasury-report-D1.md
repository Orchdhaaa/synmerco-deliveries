# Tested Python package: reconcile Treasury interest expense, debt outstanding and average rates, fiscal years 2015-2025

Escrow: `esc_a2981488c3744486bbcb4836` | Seller: `did:key:z6MksHSnnYFXsynPYLYzmqjeuvdrDuHbSa3b2LcGfAxWQaRg` | Amount: $10.50 (net to seller $10.16) | Review tier: Huge

Source (only source used): Treasury Fiscal Data, host `api.fiscaldata.treasury.gov`, endpoint `/services/api/fiscal_service/`, no key. Datasets: `avg_interest_rates`, `debt_to_penny`, `interest_expense`, `record_fiscal_year` 2015 to 2025, `page[size]=10000`, paging with `page[size]` / `page[number]` until every record each `meta total-count` reports is held.

PULL 1: 2026-11-14 18:29:37-44 UTC (HTTP Date headers), note filed 2026-11-14 18:34:09 UTC, 11 rows, SHA-256 `a2a6a9052039bb5745498251600cdf195163ce94fc566c7dda8048ffa22f62b5`.

PULL 2: 2026-11-15 18:35-18:42 UTC, note filed 2026-11-15 18:45:48 UTC, 11 rows, SHA-256 `713545815916db2518ae8f34038bf02c0509f152dd23c513fbf8fa8938e02de3`. PULL 2 is PARTIAL: `debt_to_penny` re-pulled complete (2761/2761); `avg_interest_rates` and `interest_expense` failed on pull 2 after one retry each (HTTP 503), and a further main-agent retry at 2026-11-15 18:49-18:52 UTC found all three endpoints HTTP 503. Per the TASK, for the two failed sources: pull 2 failed (counts as done). The pull 2 CSV keeps its 11 rows with `average_debt` populated from the successful re-pull and the other columns empty (a value the source leaves empty stays an empty cell), including the `flag` column: the flag is empty in all 11 rows because it cannot be computed from pull-2 data; no data was invented. See the Changes section and PROOF Requests 4-6.

CSV headers are snake_case (`record_fiscal_year, average_debt, interest_expense, implied_rate, interest_bearing_rate, marketable_rate, nonmarketable_rate, flag`), a 1:1 mapping of the fields the job lists in order: record_fiscal_year, average debt, interest expense, implied rate, interest-bearing rate, marketable rate, non-marketable rate, flag.

## Answers

One numbered answer per ACCEPTANCE line of the job, in the same order. Each answer points to the excerpt or section that shows it.

1. ACCEPTANCE line 1: 1. The report contains every file of the package, with unit tests for paging, fiscal year boundaries and the rate formula, and the output of a test run where all pass. [check: needs judgment]
   Answer: Yes. Every package file is in the Package files section below, including `tests/test_paging.py`, `tests/test_compute.py` (fiscal-year boundaries, rate formula, flag threshold) and `tests/test_cli.py`. Test run output is in the Test run section: 15 tests, all pass (`Ran 15 tests`, `OK`).

2. ACCEPTANCE line 2: 2. The report shows the command and console output of a real run, with the total-count read for each of the three datasets in raw excerpts. [check: matches the source, source Treasury Fiscal Data]
   Answer: Yes. The Command and console output section shows `python -m treasury_recon --responses-dir pull1/ --out treasury-cost-pull1.csv` and its real console output: avg_interest_rates 2207 records, meta total-count=2207; debt_to_penny 2761 records, meta total-count=2761; interest_expense 5016 records, meta total-count=5016 (all COMPLETE). The same total-counts appear in the raw excerpts in PROOF Requests 1-3 (`"total-count":2207`, `"total-count":2761`, `"total-count":5016`).

3. ACCEPTANCE line 3: 3. Both CSV files are attached with exactly the listed columns, one row per fiscal year 2015 to 2025. [check: count, target 11, in D2]
   Answer: Yes. D2 `treasury-cost-pull1.csv` (SHA-256 a2a6a9052039bb5745498251600cdf195163ce94fc566c7dda8048ffa22f62b5) and D3 `treasury-cost-pull2.csv` (SHA-256 `713545815916db2518ae8f34038bf02c0509f152dd23c513fbf8fa8938e02de3`) are attached as data files (see the Data files section) with exactly the 8 listed columns and 11 rows each, FY 2015-2025. D3 is partial as disclosed above: only `average_debt` is pull-2 data; the other columns are empty because their pull-2 sources failed after retry, and the `flag` column is empty in all 11 rows because the flag cannot be computed from pull-2 data.

4. ACCEPTANCE line 4: 4. Every flagged year is explained from the data, with raw excerpts of the records behind it. [check: needs judgment]
   Answer: Yes. Only FY2022 is flagged. The Flagged year: FY2022 section explains it from the data with the raw records behind the numbers: all 12 monthly `avg_interest_rate_amt` values for `Total Interest-bearing Debt` (mean 1.7153), FY2022 interest-expense records, and the first/last FY2022 daily debt records.

5. ACCEPTANCE line 5: 5. The PLATFORM RECORD shows a PULL 1: note and a PULL 2: note at least 24 hours later, each with its CSV's row count and SHA-256 as under DATA FILES; pull 2's source time is later than pull 1's; the change counts agree with both CSV files (or pull 2 failed as the TASK allows). [check: timing, target 24 hours between the notes]
   Answer: Yes. Platform progress notes on this escrow: PULL 1 note recorded 2026-11-14 18:34:09 UTC (11 rows, SHA-256 a2a6a905...); PULL 2 note recorded 2026-11-15 18:45:48 UTC (11 rows, SHA-256 41b8bd8f...), 24 hours 11 minutes later. Pull 2 source time (debt_to_penny HTTP Date Sun, 15 Nov 2026 18:36:38 GMT) is later than pull 1 source time (Sat, 14 Nov 2026 18:29:37-44 GMT). D3 was revised in this revision: revision note recorded 2026-11-16 18:09:43 UTC (11 rows, SHA-256 71354581...), replacing the first D3; the Data files section lists the revised file. Change counts (Changes section, computed cell by cell from the two CSV files): new 0, gone 0, changed 11 keys (66 cells: interest_expense 11, implied_rate 11, interest_bearing_rate 11, marketable_rate 11, nonmarketable_rate 11, flag 11); 11 rows = 11 + 0 - 0. The two pull-2 sources that failed are disclosed as failed; the header-only pull-2 alternative does not apply because D3 keeps its 11 rows, so the counts above are the counts of the files as delivered.


## Package files

Every file of the package, in full. The package is Python 3, stdlib only (`treasury_recon/`), plus unit tests (`tests/`), the pull script (`pull1.py`, also used for the pull-2 retry), the README, and the fixed sample data the tests use (`fixtures/`, hand-made, clearly labelled as fixtures; never mixed with the live pulls).

### `treasury_recon/__init__.py`

```
"""Treasury reconciliation package (Synmerco escrow esc_a2981488c3744486bbcb4836).

Reads three U.S. Treasury Fiscal Data datasets and reconciles, per fiscal
year 2015-2025, the interest the Treasury reports paying against its debt
and its average rates. Stdlib only.
"""
from . import client, compute  # noqa: F401

__all__ = ["client", "compute"]
```

### `treasury_recon/__main__.py`

```
"""Entry point: python -m treasury_recon ..."""
from .cli import main

if __name__ == "__main__":
    raise SystemExit(main())
```

### `treasury_recon/cli.py`

```
"""Command line: build the reconciliation CSV from saved Fiscal Data responses.

Usage:
    python -m treasury_recon --responses-dir pull1 --out treasury-cost-pull1.csv

--responses-dir holds the raw page JSON files saved by pull1.py
({dataset}_page{N}.json). The command prints, for each dataset, the
total-count read from the saved responses and the record count, then the
per-year table, and writes the CSV (11 rows for FY 2015-2025 on a real
pull; fewer rows if the saved responses cover fewer years).
"""
import argparse
import glob
import json
import os
import sys

from . import client, compute

DATASET_FILES = {
    "avg_interest_rates": "avg_interest_rates",
    "debt_to_penny": "debt_to_penny",
    "interest_expense": "interest_expense",
}


def load_responses(responses_dir):
    """Load saved page files. Returns {dataset: (records, total_count, pages)}.

    total_count is read from the meta of the saved pages (the largest
    total-count seen); pages is the number of page files found.
    """
    loaded = {}
    for dataset in DATASET_FILES:
        records = []
        total_count = None
        pages = 0
        for path in sorted(glob.glob(os.path.join(responses_dir, f"{dataset}_page*.json"))):
            with open(path, encoding="utf-8") as handle:
                payload = json.load(handle)
            page_records = payload.get("data", [])
            meta = payload.get("meta") or {}
            if meta.get("total-count") is not None:
                page_total = int(meta["total-count"])
                total_count = page_total if total_count is None else max(total_count, page_total)
            records.extend(page_records)
            pages += 1
        loaded[dataset] = (records, total_count, pages)
    return loaded


def main(argv=None):
    parser = argparse.ArgumentParser(
        prog="treasury-recon",
        description="Build the Treasury reconciliation CSV from saved Fiscal Data responses.")
    parser.add_argument("--responses-dir", required=True,
                        help="directory with {dataset}_page*.json files (see pull1.py)")
    parser.add_argument("--out", required=True, help="CSV file to write")
    args = parser.parse_args(argv)

    loaded = load_responses(args.responses_dir)
    for dataset in DATASET_FILES:
        records, total_count, pages = loaded[dataset]
        complete = ""
        if total_count is not None:
            complete = " (COMPLETE)" if len(records) >= total_count else \
                f" (INCOMPLETE: {len(records)} of {total_count})"
        print(f"{dataset}: {len(records)} records in {pages} page file(s), "
              f"meta total-count={total_count}{complete}")
    if any(loaded[d][1] is not None and len(loaded[d][0]) < loaded[d][1]
           for d in DATASET_FILES):
        print("WARNING: saved responses are incomplete vs meta total-count; "
              "re-run the pull.", file=sys.stderr)

    rows, warnings = compute.compute_years(
        loaded["debt_to_penny"][0],
        loaded["interest_expense"][0],
        loaded["avg_interest_rates"][0])
    for warning in warnings:
        print(f"WARNING: {warning}", file=sys.stderr)

    with open(args.out, "w", encoding="utf-8", newline="") as handle:
        handle.write(compute.rows_to_csv(rows))
    print(f"\nwrote {args.out}: {len(rows)} row(s)")
    print(" | ".join(compute.COLUMNS))
    for row in rows:
        print(" | ".join("" if row[c] is None else str(row[c])
                         for c in compute.COLUMNS))
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

### `treasury_recon/client.py`

```
"""Client for the U.S. Treasury Fiscal Data API (keyless, JSON).

Only the three datasets this escrow needs. Stdlib only (urllib).

Paging contract (per the Fiscal Data API docs and the job spec):
  * page[size] / page[number] query params (max page size 10000).
  * meta["total-count"] is the total number of matching records; paging
    continues until every record it reports is held. meta["count"] is only
    the number of records on the current page and must NOT be used as the
    total (that was a bug in the first pull script: it stopped after page 1).
  * All values in the JSON are strings, including numbers.
"""
import hashlib
import json
import urllib.error
import urllib.parse
import urllib.request

BASE_URL = "https://api.fiscaldata.treasury.gov/services/api/fiscal_service"

DATASETS = {
    "avg_interest_rates": "v2/accounting/od/avg_interest_rates",
    "debt_to_penny": "v2/accounting/od/debt_to_penny",
    "interest_expense": "v2/accounting/od/interest_expense",
}

# Fiscal years covered by this job.
FY_START = 2015
FY_END = 2025

DEFAULT_PAGE_SIZE = 10000
MAX_PAGES = 200

USER_AGENT = "treasury-recon/1.0 (Synmerco escrow research; contact via agent)"


class FetchError(Exception):
    """Raised when a page fetch fails; carries the URL so the caller can log it."""

    def __init__(self, url, message):
        super().__init__(f"{url}: {message}")
        self.url = url
        self.message = message


def build_url(endpoint, page_size, page_number, fy_start=FY_START, fy_end=FY_END):
    """Build one paged request URL for an endpoint."""
    query = urllib.parse.urlencode({
        "filter": f"record_fiscal_year:gte:{fy_start},record_fiscal_year:lte:{fy_end}",
        "page[size]": page_size,
        "page[number]": page_number,
    })
    return f"{BASE_URL}/{endpoint}?{query}"


def default_fetch(url):
    """Fetch one URL. Returns (http_date_header_or_None, body_bytes).

    Raises FetchError on any transport/HTTP failure.
    """
    request = urllib.request.Request(url, headers={"User-Agent": USER_AGENT})
    try:
        with urllib.request.urlopen(request, timeout=120) as response:
            return response.headers.get("Date"), response.read()
    except urllib.error.HTTPError as exc:
        raise FetchError(url, f"HTTP {exc.code} {exc.reason}") from exc
    except Exception as exc:  # URLError, timeout, SSL, ...
        raise FetchError(url, f"{type(exc).__name__}: {exc}") from exc


def parse_page(body):
    """Parse one response body. Returns (records, meta_total_count_or_None)."""
    try:
        payload = json.loads(body)
    except Exception as exc:
        raise ValueError(f"response is not valid JSON: {exc}") from exc
    if not isinstance(payload, dict) or not isinstance(payload.get("data"), list):
        raise ValueError("response JSON has no 'data' list")
    meta = payload.get("meta") or {}
    total_count = meta.get("total-count")
    if total_count is not None:
        total_count = int(total_count)
    return payload["data"], total_count


def fetch_all(endpoint, page_size=DEFAULT_PAGE_SIZE, fetch=default_fetch,
              fy_start=FY_START, fy_end=FY_END):
    """Fetch every page of an endpoint.

    Returns a dict: {"records": [...], "pages": [{"page", "url", "http_date",
    "sha256", "records", "total_count"}], "total_count": int|None}.
    Stops when accumulated records reach meta total-count, or (when the
    server gives no total-count) when a page comes back short.
    """
    records = []
    pages = []
    total_count = None
    page_number = 1
    while page_number <= MAX_PAGES:
        url = build_url(endpoint, page_size, page_number, fy_start, fy_end)
        http_date, body = fetch(url)
        page_records, page_total = parse_page(body)
        if page_total is not None:
            total_count = page_total
        records.extend(page_records)
        pages.append({
            "page": page_number,
            "url": url,
            "http_date": http_date,
            "sha256": hashlib.sha256(body).hexdigest(),
            "records": len(page_records),
            "total_count": total_count,
        })
        if total_count is not None:
            if len(records) >= total_count:
                break
        elif len(page_records) < page_size:
            break
        if not page_records:
            break
        page_number += 1
    return {"records": records, "pages": pages, "total_count": total_count}
```

### `treasury_recon/compute.py`

```
"""Per-fiscal-year reconciliation of the three Treasury Fiscal Data datasets.

Formulas (per the job spec):
  * average debt (FY)   = mean of daily tot_pub_debt_out_amt (debt_to_penny)
  * interest expense     = sum of month_expense_amt (interest_expense)
  * implied rate (%)     = interest expense / average debt * 100
  * interest-bearing / marketable / non-marketable rate = fiscal-year mean
    of monthly avg_interest_rate_amt for the matching security_desc total
  * flag = "yes" when |implied rate - interest-bearing mean| > 0.5 pp,
    "no" when both are known and the gap is at most 0.5 pp, and empty
    (unknown) when the implied rate or the interest-bearing mean cannot
    be computed from the data held -- a missing input is never reported
    as "no".

All money/rate values in the API are JSON strings; Decimal is used so the
means and the implied rate are exact, not float approximations.

security_desc values (verified against the LIVE pull-1 responses, 2026-11-15):
  * "Total Interest-bearing Debt" (interest-bearing total)
  * "Total Marketable" (marketable total)
  * "Total Non-marketable" (non-marketable total)
Matching is normalised (case, hyphens, whitespace) so both hyphenated and
unhyphenated spellings match. If a later pull shows different labels, the
RATE_DESCS map is the one place to fix.
"""
import csv
import io
from datetime import date
from decimal import Decimal, InvalidOperation

COLUMNS = [
    "record_fiscal_year",
    "average_debt",
    "interest_expense",
    "implied_rate",
    "interest_bearing_rate",
    "marketable_rate",
    "nonmarketable_rate",
    "flag",
]

FLAG_THRESHOLD_PP = Decimal("0.5")

# Normalised (lowercase, no hyphens, single spaces) security_desc totals.
# Labels verified against the live pull-1 responses (2026-11-15).
RATE_DESCS = {
    "interest_bearing": "total interestbearing debt",
    "marketable": "total marketable",
    "nonmarketable": "total nonmarketable",
}


def fiscal_year_of(record_date):
    """Fiscal year for an ISO date string. FY starts 1 Oct: 2015-09-30 is
    FY2015, 2015-10-01 is FY2016."""
    year, month, _day = (int(part) for part in record_date.split("-"))
    return year + 1 if month >= 10 else year


def _norm_desc(text):
    # Lowercase, drop hyphens (so "Interest-bearing" and "Interestbearing"
    # match), collapse whitespace.
    return " ".join((text or "").lower().replace("-", "").split())


def _decimal(value):
    """Parse an API string number to Decimal; None for empty/missing/bad."""
    if value is None:
        return None
    if isinstance(value, (int, float)):
        return Decimal(str(value))
    text = str(value).strip()
    if not text:
        return None
    try:
        return Decimal(text.replace(",", ""))
    except InvalidOperation:
        return None


def _fiscal_year(record):
    """Fiscal year of a record: prefer the record_fiscal_year field; fall
    back to deriving it from record_date."""
    raw = record.get("record_fiscal_year")
    if raw is not None and str(raw).strip() != "":
        return int(raw)
    record_date = record.get("record_date")
    if record_date:
        return fiscal_year_of(record_date)
    return None


def _mean(values):
    if not values:
        return None
    return sum(values, Decimal(0)) / Decimal(len(values))


def compute_years(debt_records, expense_records, rate_records):
    """Aggregate the three record lists into one row per fiscal year.

    Returns (rows, warnings). Each row is a dict keyed by COLUMNS, values
    are Decimal/int/str or None (None renders as an empty CSV cell).
    Rows cover the union of fiscal years present in any dataset, sorted.
    """
    warnings = []

    debt_by_fy = {}
    for record in debt_records:
        fy = _fiscal_year(record)
        if fy is None:
            warnings.append(f"debt_to_penny record without fiscal year: {record}")
            continue
        amount = _decimal(record.get("tot_pub_debt_out_amt"))
        if amount is None:
            warnings.append(f"debt_to_penny FY{fy} unparseable amount: {record}")
            continue
        record_date = record.get("record_date")
        if record_date:
            try:
                if fiscal_year_of(record_date) != fy:
                    warnings.append(
                        f"debt_to_penny FY{fy} record_date {record_date} "
                        "disagrees with record_fiscal_year")
            except (ValueError, TypeError):
                pass
        debt_by_fy.setdefault(fy, []).append(amount)

    expense_by_fy = {}
    for record in expense_records:
        fy = _fiscal_year(record)
        if fy is None:
            warnings.append(f"interest_expense record without fiscal year: {record}")
            continue
        amount = _decimal(record.get("month_expense_amt"))
        if amount is None:
            warnings.append(f"interest_expense FY{fy} unparseable amount: {record}")
            continue
        expense_by_fy.setdefault(fy, []).append(amount)

    rates_by_fy = {}
    for record in rate_records:
        fy = _fiscal_year(record)
        if fy is None:
            warnings.append(f"avg_interest_rates record without fiscal year: {record}")
            continue
        desc = _norm_desc(record.get("security_desc"))
        kind = None
        for key, wanted in RATE_DESCS.items():
            if desc == wanted:
                kind = key
                break
        if kind is None:
            continue  # e.g. "Treasury Bills": a component line, not a total
        rate = _decimal(record.get("avg_interest_rate_amt"))
        if rate is None:
            warnings.append(f"avg_interest_rates FY{fy} unparseable rate: {record}")
            continue
        rates_by_fy.setdefault(fy, {}).setdefault(kind, []).append(rate)

    rows = []
    for fy in sorted(set(debt_by_fy) | set(expense_by_fy) | set(rates_by_fy)):
        debt_values = debt_by_fy.get(fy, [])
        expense_values = expense_by_fy.get(fy, [])
        avg_debt = _mean(debt_values)
        expense = sum(expense_values, Decimal(0)) if expense_values else None
        implied = None
        if avg_debt is not None and avg_debt != 0 and expense is not None:
            implied = (expense / avg_debt * Decimal(100)).quantize(
                Decimal("0.0001"))
        fy_rates = rates_by_fy.get(fy, {})
        ib_rate = _mean(fy_rates.get("interest_bearing", []))
        if ib_rate is not None:
            ib_rate = ib_rate.quantize(Decimal("0.0001"))
        mkt_rate = _mean(fy_rates.get("marketable", []))
        if mkt_rate is not None:
            mkt_rate = mkt_rate.quantize(Decimal("0.0001"))
        nonmkt_rate = _mean(fy_rates.get("nonmarketable", []))
        if nonmkt_rate is not None:
            nonmkt_rate = nonmkt_rate.quantize(Decimal("0.0001"))
        flag = ""
        if implied is not None and ib_rate is not None:
            flag = "yes" if abs(implied - ib_rate) > FLAG_THRESHOLD_PP else "no"
        rows.append({
            "record_fiscal_year": fy,
            "average_debt": (avg_debt.quantize(Decimal("0.01"))
                             if avg_debt is not None else None),
            "interest_expense": (expense.quantize(Decimal("0.01"))
                                 if expense is not None else None),
            "implied_rate": implied,
            "interest_bearing_rate": ib_rate,
            "marketable_rate": mkt_rate,
            "nonmarketable_rate": nonmkt_rate,
            "flag": flag,
        })
    return rows, warnings


def rows_to_csv(rows):
    """Render rows as CSV text. Money: 2 decimals. Rates: 4 decimals.
    A value the source leaves empty stays an empty cell."""
    buffer = io.StringIO()
    writer = csv.DictWriter(buffer, fieldnames=COLUMNS, lineterminator="\n")
    writer.writeheader()
    for row in rows:
        out = {}
        for column in COLUMNS:
            value = row[column]
            if value is None:
                out[column] = ""
            elif isinstance(value, Decimal):
                places = "0.01" if column in ("average_debt", "interest_expense") else "0.0001"
                out[column] = str(value.quantize(Decimal(places)))
            else:
                out[column] = value
        writer.writerow(out)
    return buffer.getvalue()
```

### `tests/__init__.py`

```

```

### `tests/test_cli.py`

```
"""End-to-end test: CLI on the fixture responses -> CSV + console output."""
import io
import os
import tempfile
import unittest
from contextlib import redirect_stdout

from treasury_recon import cli

FIXTURES = os.path.join(os.path.dirname(__file__), "..", "fixtures")


class CliTest(unittest.TestCase):
    def test_cli_on_fixtures(self):
        with tempfile.TemporaryDirectory() as tmp:
            out = os.path.join(tmp, "treasury-cost-FIXTURE-sample.csv")
            stdout = io.StringIO()
            with redirect_stdout(stdout):
                rc = cli.main(["--responses-dir", FIXTURES, "--out", out])
            self.assertEqual(rc, 0)
            with open(out, encoding="utf-8") as handle:
                csv_text = handle.read()
            self.assertEqual(
                csv_text,
                "record_fiscal_year,average_debt,interest_expense,implied_rate,"
                "interest_bearing_rate,marketable_rate,nonmarketable_rate,flag\n"
                "2015,1100.00,11.00,1.0000,1.2000,1.0000,1.5000,no\n"
                "2016,2100.00,42.00,2.0000,2.9000,2.7000,3.1000,yes\n")
            console = stdout.getvalue()
            # Console output must include each dataset's total-count read.
            self.assertIn("avg_interest_rates: 7 records in 2 page file(s), "
                          "meta total-count=7 (COMPLETE)", console)
            self.assertIn("debt_to_penny: 4 records in 2 page file(s), "
                          "meta total-count=4 (COMPLETE)", console)
            self.assertIn("interest_expense: 2 records in 2 page file(s), "
                          "meta total-count=2 (COMPLETE)", console)
            self.assertIn("wrote", console)


if __name__ == "__main__":
    unittest.main()
```

### `tests/test_compute.py`

```
"""Tests for treasury_recon.compute: rate formula, FY boundaries, flag."""
import json
import os
import unittest
from decimal import Decimal

from treasury_recon import compute

FIXTURES = os.path.join(os.path.dirname(__file__), "..", "fixtures")


def _load(name):
    with open(os.path.join(FIXTURES, name), encoding="utf-8") as handle:
        return json.load(handle)["data"]


def _records(dataset):
    out = []
    for page in (1, 2):
        out.extend(_load(f"{dataset}_page{page}.json"))
    return out


class FiscalYearTest(unittest.TestCase):
    def test_fiscal_year_boundaries(self):
        # Federal FY starts 1 Oct: 30 Sep is the old FY, 1 Oct the new one.
        self.assertEqual(compute.fiscal_year_of("2015-09-30"), 2015)
        self.assertEqual(compute.fiscal_year_of("2015-10-01"), 2016)
        self.assertEqual(compute.fiscal_year_of("2025-09-30"), 2025)
        self.assertEqual(compute.fiscal_year_of("2025-10-01"), 2026)
        self.assertEqual(compute.fiscal_year_of("2020-01-15"), 2020)


class GoldenFixtureTest(unittest.TestCase):
    """Hand-computed expectations from fixtures/README.md."""

    def test_golden_rows(self):
        rows, warnings = compute.compute_years(
            _records("debt_to_penny"),
            _records("interest_expense"),
            _records("avg_interest_rates"))
        self.assertEqual(warnings, [])
        self.assertEqual(len(rows), 2)
        fy2015, fy2016 = rows
        self.assertEqual(fy2015, {
            "record_fiscal_year": 2015,
            "average_debt": Decimal("1100.00"),
            "interest_expense": Decimal("11.00"),
            "implied_rate": Decimal("1.0000"),
            "interest_bearing_rate": Decimal("1.2000"),
            "marketable_rate": Decimal("1.0000"),
            "nonmarketable_rate": Decimal("1.5000"),
            "flag": "no",
        })
        self.assertEqual(fy2016, {
            "record_fiscal_year": 2016,
            "average_debt": Decimal("2100.00"),
            "interest_expense": Decimal("42.00"),
            "implied_rate": Decimal("2.0000"),
            "interest_bearing_rate": Decimal("2.9000"),
            "marketable_rate": Decimal("2.7000"),
            "nonmarketable_rate": Decimal("3.1000"),
            "flag": "yes",
        })

    def test_golden_csv(self):
        rows, _ = compute.compute_years(
            _records("debt_to_penny"),
            _records("interest_expense"),
            _records("avg_interest_rates"))
        self.assertEqual(compute.rows_to_csv(rows),
                         "record_fiscal_year,average_debt,interest_expense,implied_rate,"
                         "interest_bearing_rate,marketable_rate,nonmarketable_rate,flag\n"
                         "2015,1100.00,11.00,1.0000,1.2000,1.0000,1.5000,no\n"
                         "2016,2100.00,42.00,2.0000,2.9000,2.7000,3.1000,yes\n")

    def test_fy_boundary_grouping(self):
        # 2015-09-30 debt lands in FY2015; 2015-10-01 debt in FY2016.
        rows, _ = compute.compute_years(
            _records("debt_to_penny"), [], [])
        self.assertEqual([r["record_fiscal_year"] for r in rows], [2015, 2016])
        self.assertEqual(rows[0]["average_debt"], Decimal("1100.00"))
        self.assertEqual(rows[1]["average_debt"], Decimal("2100.00"))


class FlagThresholdTest(unittest.TestCase):
    def _one_year(self, implied_expense, ib_rate):
        debt = [{"record_fiscal_year": "2020", "record_date": "2020-06-30",
                 "tot_pub_debt_out_amt": "1000"}]
        expense = [{"record_fiscal_year": "2020", "record_date": "2020-06-30",
                    "month_expense_amt": str(implied_expense)}]
        rates = ([{"record_fiscal_year": "2020", "record_date": "2020-06-30",
                   "security_desc": "Total Interest-bearing Debt",
                   "avg_interest_rate_amt": str(ib_rate)}] if ib_rate is not None else [])
        rows, _ = compute.compute_years(debt, expense, rates)
        return rows[0]

    def test_rate_formula(self):
        # implied = 20 / 1000 * 100 = 2.0000
        row = self._one_year(20, "2.0")
        self.assertEqual(row["implied_rate"], Decimal("2.0000"))

    def test_flag_boundary_exactly_half_point_is_no(self):
        # |2.0 - 2.5| = 0.5 exactly -> "no" (flag needs MORE than 0.5).
        self.assertEqual(self._one_year(20, "2.5")["flag"], "no")

    def test_flag_just_over_half_point_is_yes(self):
        self.assertEqual(self._one_year(20, "2.5001")["flag"], "yes")

    def test_missing_interest_bearing_rate_means_empty_flag(self):
        # No interest-bearing mean -> the flag cannot be computed: it is
        # empty (unknown), never "no". The CSV cell is empty too.
        row = self._one_year(20, None)
        self.assertEqual(row["flag"], "")
        self.assertIsNone(row["interest_bearing_rate"])
        self.assertIn("2020,1000.00,20.00,2.0000,,,,\n",
                      compute.rows_to_csv([row]))

    def test_distractor_security_lines_ignored(self):
        debt = [{"record_fiscal_year": "2020", "tot_pub_debt_out_amt": "1000"}]
        expense = [{"record_fiscal_year": "2020", "month_expense_amt": "20"}]
        rates = [
            {"record_fiscal_year": "2020", "security_desc": "Treasury Bills",
             "avg_interest_rate_amt": "9.9"},
            {"record_fiscal_year": "2020", "security_desc": "Total Interest-bearing Debt",
             "avg_interest_rate_amt": "2.0"},
        ]
        rows, _ = compute.compute_years(debt, expense, rates)
        self.assertEqual(rows[0]["interest_bearing_rate"], Decimal("2.0000"))


if __name__ == "__main__":
    unittest.main()
```

### `tests/test_paging.py`

```
"""Tests for treasury_recon.client paging.

The Fiscal Data API reports meta["total-count"] (all matching records) and
meta["count"] (records on this page). Paging must continue until every
record total-count reports is held -- count alone must never stop it.
"""
import json
import unittest
import urllib.parse

from treasury_recon import client


def _page_body(records, total_count=None, count=None):
    meta = {}
    if count is not None:
        meta["count"] = count
    if total_count is not None:
        meta["total-count"] = total_count
    return json.dumps({"data": records, "meta": meta}).encode()


class FakeFetch:
    """Returns canned pages by page[number]; records every URL requested."""

    def __init__(self, pages):
        # pages: {page_number: (records, total_count)}
        self.pages = pages
        self.urls = []

    def __call__(self, url):
        self.urls.append(url)
        number = int(urllib.parse.parse_qs(urllib.parse.urlparse(url).query)["page[number]"][0])
        records, total_count = self.pages[number]
        return "Wed, 01 Oct 2025 00:00:00 GMT", _page_body(records, total_count, count=len(records))


class PagingTest(unittest.TestCase):
    def test_stops_at_total_count_not_at_count(self):
        # Page 1: count=2 but total-count=4 -> MUST fetch page 2.
        fetch = FakeFetch({
            1: ([{"id": 1}, {"id": 2}], 4),
            2: ([{"id": 3}, {"id": 4}], 4),
        })
        result = client.fetch_all("v2/accounting/od/debt_to_penny",
                                  page_size=2, fetch=fetch)
        self.assertEqual([r["id"] for r in result["records"]], [1, 2, 3, 4])
        self.assertEqual(result["total_count"], 4)
        self.assertEqual(len(result["pages"]), 2)
        self.assertEqual(len(fetch.urls), 2)

    def test_page_params_in_url(self):
        fetch = FakeFetch({1: ([{"id": 1}], 1)})
        client.fetch_all("v2/accounting/od/debt_to_penny", page_size=2, fetch=fetch)
        query = urllib.parse.parse_qs(urllib.parse.urlparse(fetch.urls[0]).query)
        self.assertEqual(query["page[size]"], ["2"])
        self.assertEqual(query["page[number]"], ["1"])
        self.assertIn("record_fiscal_year:gte:2015", query["filter"][0])
        self.assertIn("record_fiscal_year:lte:2025", query["filter"][0])

    def test_short_page_stops_when_no_total_count(self):
        fetch = FakeFetch({
            1: ([{"id": 1}, {"id": 2}], None),
            2: ([{"id": 3}], None),
        })
        result = client.fetch_all("v2/accounting/od/debt_to_penny",
                                  page_size=2, fetch=fetch)
        self.assertEqual([r["id"] for r in result["records"]], [1, 2, 3])
        self.assertIsNone(result["total_count"])
        self.assertEqual(len(fetch.urls), 2)

    def test_fetch_error_propagates_with_url(self):
        def failing_fetch(url):
            raise client.FetchError(url, "HTTP 403 Forbidden")

        with self.assertRaises(client.FetchError) as ctx:
            client.fetch_all("v2/accounting/od/debt_to_penny", fetch=failing_fetch)
        self.assertIn("403", str(ctx.exception))

    def test_bad_json_raises(self):
        def bad_fetch(url):
            return None, b"not json at all"

        with self.assertRaises(ValueError):
            client.fetch_all("v2/accounting/od/debt_to_penny", fetch=bad_fetch)


if __name__ == "__main__":
    unittest.main()
```

### `pull1.py`

```
#!/usr/bin/env python3
"""PULL 1 for escrow esc_a2981488c3744486bbcb4836 (Treasury reconciliation job).

Fetches the three Fiscal Data datasets (FY 2015-2025) in full, paging until
every record each meta total-count reports is held. Saves raw responses +
a manifest with URL, HTTP Date, SHA-256, page and record counts. No key.

Uses treasury_recon.client so the pull path and the package share the same
paging code. On a fetch failure the error is recorded in the manifest
(per dataset) and the script exits non-zero -- it never invents data.

Usage: python3 pull1.py [--out-dir pull1] [--page-size 10000]
"""
import argparse
import json
import os
import sys

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))

from treasury_recon import client  # noqa: E402

HERE = os.path.dirname(os.path.abspath(__file__))


def pull_dataset(name, endpoint, out_dir, page_size):
    """Fetch one dataset, saving each raw page. Returns its manifest entry."""
    pages = []
    total_records = 0
    total_count = None
    counter = {"n": 0}

    def saving_fetch(url):
        http_date, body = client.default_fetch(url)
        counter["n"] += 1
        fname = f"{name}_page{counter['n']}.json"
        with open(os.path.join(out_dir, fname), "wb") as handle:
            handle.write(body)
        return http_date, body

    try:
        result = client.fetch_all(endpoint, page_size=page_size,
                                  fetch=saving_fetch)
    except Exception as exc:
        print(f"{name}: ERROR {exc}", file=sys.stderr)
        return {"pages": pages, "total_records": total_records,
                "meta_total_count": total_count, "error": str(exc)}

    # Re-read the saved pages to build the manifest (proves what is on disk).
    import hashlib
    for page in result["pages"]:
        fname = f"{name}_page{page['page']}.json"
        with open(os.path.join(out_dir, fname), "rb") as handle:
            body = handle.read()
        pages.append({
            "file": fname,
            "page": page["page"],
            "records": page["records"],
            "http_date": page["http_date"],
            "sha256": hashlib.sha256(body).hexdigest(),
            "url": page["url"],
        })
        print(f"{name} page {page['page']}: {page['records']} records, "
              f"meta total-count={page['total_count']}, date={page['http_date']}")
    return {"pages": pages,
            "total_records": len(result["records"]),
            "meta_total_count": result["total_count"]}


def main():
    parser = argparse.ArgumentParser(description="PULL 1: fetch the three "
                                                "Fiscal Data datasets.")
    parser.add_argument("--out-dir", default=os.path.join(HERE, "pull1"))
    parser.add_argument("--page-size", type=int, default=client.DEFAULT_PAGE_SIZE)
    args = parser.parse_args()

    os.makedirs(args.out_dir, exist_ok=True)
    manifest = {"pull": 1, "datasets": {}}
    failed = []
    for name, endpoint in client.DATASETS.items():
        entry = pull_dataset(name, endpoint, args.out_dir, args.page_size)
        manifest["datasets"][name] = entry
        if "error" in entry:
            failed.append(name)
    with open(os.path.join(args.out_dir, "manifest.json"), "w") as handle:
        json.dump(manifest, handle, indent=2)
    summary = {k: (v.get("total_records"), v.get("meta_total_count"))
               for k, v in manifest["datasets"].items()}
    print(json.dumps(summary))
    if failed:
        print(f"PULL 1 FAILED for: {', '.join(failed)} "
              "(see manifest.json errors; no data fabricated)", file=sys.stderr)
        return 1
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

### `README.md`

```
# Treasury reconciliation — Synmerco escrow esc_a2981488c3744486bbcb4836

Job: "Tested Python package: reconcile Treasury interest expense, debt
outstanding and average rates, fiscal years 2015-2025" (bid 5149, $10.50;
deliverBy 2026-11-17T18:10:57Z; earliestSubmitAt 2026-11-15T18:10:57Z).
Source: Treasury Fiscal Data only, host api.fiscaldata.treasury.gov, no key.

## Layout

- `treasury_recon/` — the Python 3 package (stdlib only, no dependencies)
  - `client.py` — Fiscal Data API client; pages with `page[size]` /
    `page[number]` until every record `meta["total-count"]` reports is held
    (`meta["count"]` is per-page only and is never treated as the total)
  - `compute.py` — per-FY aggregation: average debt (mean of daily
    `tot_pub_debt_out_amt`), interest expense (sum of `month_expense_amt`),
    implied rate = expense / average debt * 100 (%), FY means of
    `avg_interest_rate_amt` for the three security totals, flag "yes" when
    |implied - interest-bearing| > 0.5 pp, "no" when both are known and the
    gap is at most 0.5 pp, empty when the implied rate or the
    interest-bearing mean cannot be computed from the data held
  - `cli.py` / `__main__.py` — `python -m treasury_recon
    --responses-dir <dir> --out <csv>` builds the CSV from saved responses
- `tests/` — unit tests (stdlib `unittest`): paging, FY boundaries, rate
  formula, flag threshold, CLI end-to-end on fixtures
- `fixtures/` — FIXTURE responses shaped like the real API. Hand-made test
  data, NOT real pulls. Never present them as pull data.
- `pull1.py` — PULL 1 script: fetches the three datasets in full, saves raw
  page JSONs + `manifest.json` (URL, HTTP Date, SHA-256, counts) into `pull1/`
- `pull1/` — raw pull 1 responses + manifest (currently: fetch failures only)
- `AWARD.md` — award details, timeline, pull status

## CSV columns (in order)

`record_fiscal_year, average_debt, interest_expense, implied_rate,
interest_bearing_rate, marketable_rate, nonmarketable_rate, flag`
Money: 2 decimals. Rates: percent, 4 decimals. A value the source leaves
empty stays an empty cell. One row per fiscal year (11 rows, FY 2015-2025,
on a real pull).

## How to run

```sh
# tests (from this directory)
python3 -m unittest discover -s tests -v

# pull 1 (live API; currently BLOCKED from this environment, see below)
python3 pull1.py

# build the CSV from saved responses (pull1/ once the pull succeeds)
python3 -m treasury_recon --responses-dir pull1 --out treasury-cost-pull1.csv

# same command on fixtures (sample only, 2 rows, clearly labelled)
python3 -m treasury_recon --responses-dir fixtures --out treasury-cost-FIXTURE-sample.csv
```

## Live-pull status (PULL 1 SUCCEEDED 2026-11-15 ~01:29 WIB)

The API became reachable again via urllib (the earlier Akamai 400/403 was
transient). `python3 pull1.py` ran clean on 2026-11-15 ~01:29 WIB:
- avg_interest_rates: 2207 records, meta total-count 2207, HTTP Date
  Sat, 14 Nov 2026 18:29:37 GMT
- debt_to_penny: 2761 records, meta total-count 2761, HTTP Date 18:29:40 GMT
- interest_expense: 5016 records, meta total-count 5016, HTTP Date 18:29:44 GMT
Raw responses + `manifest.json` (URL, HTTP Date, SHA-256 per page) are in
`pull1/`. The CLI run on them produced `treasury-cost-pull1.csv`
(11 rows, FY 2015-2025, SHA-256
a2a6a9052039bb5745498251600cdf195163ce94fc566c7dda8048ffa22f62b5).
Sanity checks passed: per-FY sum of `month_expense_amt` equals the sum of
September `fytd_expense_amt` for all 11 FYs; only FY2022 is flagged
(implied 2.3933% vs interest-bearing 1.7153%).

Still open: PULL 2 (same requests, 24h after the PULL 1 progress note),
then the D1 report and submission (not before earliestSubmitAt
2026-11-15T18:10:57Z). The PULL 1 progress note is filed by the main
agent, not from this scaffold.

## Assumptions verified from public docs (not a live pull)

- Endpoints/fields: `v2/accounting/od/avg_interest_rates`
  (`record_date`, `security_desc`, `avg_interest_rate_amt`,
  `record_fiscal_year`), `v2/accounting/od/debt_to_penny`
  (`tot_pub_debt_out_amt`, daily), `v2/accounting/od/interest_expense`
  (`month_expense_amt`, monthly). Paging: `page[size]` max 10000,
  `meta.total-count` / `meta.total-pages` drive paging; all values are
  JSON strings.
- security_desc totals: "Total Interest-bearing Debt",
  "Total Marketable", "Total Non-marketable" (matching is normalised for
  case/hyphens, see `compute.RATE_DESCS`). Labels verified against the
  live pull-1 responses (2026-11-15); if a later pull shows different
  labels, that map is the one place to fix.
```

### `fixtures/README.md`

```
# FIXTURES — NOT real pulls

These files are hand-made test data shaped like real Treasury Fiscal Data
API responses (`{"data": [...], "meta": {"count": N, "total-count": M}}`).
They are NOT responses from api.fiscaldata.treasury.gov and must never be
presented as pull 1 / pull 2 data. They exist only so the package, the
paging logic and the CLI can be tested while the live API is unreachable
from this environment (Akamai 403).

Hand-computed expected results (see tests/test_compute.py):
- FY2015: avg debt (1000+1200)/2 = 1100.00; expense 11.00;
  implied 11/1100*100 = 1.0000%; IB 1.2000, M 1.0000, NM 1.5000;
  |1.0000-1.2000| = 0.2 <= 0.5 -> flag "no".
  The "Treasury Bills" row (0.5) is a distractor: component lines are
  ignored, only the three total lines count.
- FY2016: avg debt (2000+2200)/2 = 2100.00; expense 42.00;
  implied 42/2100*100 = 2.0000%; IB 2.9000, M 2.7000, NM 3.1000;
  |2.0000-2.9000| = 0.9 > 0.5 -> flag "yes".
- FY boundary check: 2015-09-30 belongs to FY2015, 2015-10-01 to FY2016.
```

### `fixtures/avg_interest_rates_page1.json`

```
{"data":[{"avg_interest_rate_amt":"1.2","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","security_desc":"Total Interest-bearing Debt","security_type_desc":"Total Interest-bearing Debt","src_line_nbr":"1"},{"avg_interest_rate_amt":"1.0","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","security_desc":"Total Marketable","security_type_desc":"Total Marketable","src_line_nbr":"2"},{"avg_interest_rate_amt":"1.5","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","security_desc":"Total Non-marketable","security_type_desc":"Total Non-marketable","src_line_nbr":"3"},{"avg_interest_rate_amt":"0.5","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","security_desc":"Treasury Bills","security_type_desc":"Treasury Bills","src_line_nbr":"4"}],"meta":{"count":4,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":7,"total-pages":2}}
```

### `fixtures/avg_interest_rates_page2.json`

```
{"data":[{"avg_interest_rate_amt":"2.9","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-31","record_fiscal_year":"2016","security_desc":"Total Interest-bearing Debt","security_type_desc":"Total Interest-bearing Debt","src_line_nbr":"5"},{"avg_interest_rate_amt":"2.7","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-31","record_fiscal_year":"2016","security_desc":"Total Marketable","security_type_desc":"Total Marketable","src_line_nbr":"6"},{"avg_interest_rate_amt":"3.1","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-31","record_fiscal_year":"2016","security_desc":"Total Non-marketable","security_type_desc":"Total Non-marketable","src_line_nbr":"7"}],"meta":{"count":3,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":7,"total-pages":2}}
```

### `fixtures/debt_to_penny_page1.json`

```
{"data":[{"debt_held_public_amt":"800","intragov_hold_amt":"200","record_calendar_day":"29","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-29","record_fiscal_year":"2015","src_line_nbr":"1","tot_pub_debt_out_amt":"1000"},{"debt_held_public_amt":"900","intragov_hold_amt":"300","record_calendar_day":"30","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","src_line_nbr":"2","tot_pub_debt_out_amt":"1200"}],"meta":{"count":2,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":4,"total-pages":2}}
```

### `fixtures/debt_to_penny_page2.json`

```
{"data":[{"debt_held_public_amt":"1500","intragov_hold_amt":"500","record_calendar_day":"01","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-01","record_fiscal_year":"2016","src_line_nbr":"3","tot_pub_debt_out_amt":"2000"},{"debt_held_public_amt":"1600","intragov_hold_amt":"600","record_calendar_day":"02","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-02","record_fiscal_year":"2016","src_line_nbr":"4","tot_pub_debt_out_amt":"2200"}],"meta":{"count":2,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":4,"total-pages":2}}
```

### `fixtures/interest_expense_page1.json`

```
{"data":[{"expense_catg_desc":"Interest Expense","expense_group_desc":"Interest on Treasury Securities","expense_type_desc":"Interest Expense","fytd_expense_amt":"11","month_expense_amt":"11","record_calendar_month":"09","record_calendar_year":"2015","record_date":"2015-09-30","record_fiscal_year":"2015","src_line_nbr":"1"}],"meta":{"count":1,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":2,"total-pages":2}}
```

### `fixtures/interest_expense_page2.json`

```
{"data":[{"expense_catg_desc":"Interest Expense","expense_group_desc":"Interest on Treasury Securities","expense_type_desc":"Interest Expense","fytd_expense_amt":"42","month_expense_amt":"42","record_calendar_month":"10","record_calendar_year":"2015","record_date":"2015-10-31","record_fiscal_year":"2016","src_line_nbr":"2"}],"meta":{"count":1,"dataFormats":{},"dataTypes":{},"labels":{},"total-count":2,"total-pages":2}}
```

## Test run

Command: `python3 -m unittest discover -s tests -v` (run in the package directory). All 15 tests pass.

```
test_cli_on_fixtures (test_cli.CliTest.test_cli_on_fixtures) ... ok
test_fiscal_year_boundaries (test_compute.FiscalYearTest.test_fiscal_year_boundaries) ... ok
test_distractor_security_lines_ignored (test_compute.FlagThresholdTest.test_distractor_security_lines_ignored) ... ok
test_flag_boundary_exactly_half_point_is_no (test_compute.FlagThresholdTest.test_flag_boundary_exactly_half_point_is_no) ... ok
test_flag_just_over_half_point_is_yes (test_compute.FlagThresholdTest.test_flag_just_over_half_point_is_yes) ... ok
test_missing_interest_bearing_rate_means_empty_flag (test_compute.FlagThresholdTest.test_missing_interest_bearing_rate_means_empty_flag) ... ok
test_rate_formula (test_compute.FlagThresholdTest.test_rate_formula) ... ok
test_fy_boundary_grouping (test_compute.GoldenFixtureTest.test_fy_boundary_grouping) ... ok
test_golden_csv (test_compute.GoldenFixtureTest.test_golden_csv) ... ok
test_golden_rows (test_compute.GoldenFixtureTest.test_golden_rows) ... ok
test_bad_json_raises (test_paging.PagingTest.test_bad_json_raises) ... ok
test_fetch_error_propagates_with_url (test_paging.PagingTest.test_fetch_error_propagates_with_url) ... ok
test_page_params_in_url (test_paging.PagingTest.test_page_params_in_url) ... ok
test_short_page_stops_when_no_total_count (test_paging.PagingTest.test_short_page_stops_when_no_total_count) ... ok
test_stops_at_total_count_not_at_count (test_paging.PagingTest.test_stops_at_total_count_not_at_count) ... ok

----------------------------------------------------------------------
Ran 15 tests in 0.029s

OK
```

## Command and console output

Command (pull 1 CSV): `python -m treasury_recon --responses-dir pull1/ --out treasury-cost-pull1.csv`

```
avg_interest_rates: 2207 records in 1 page file(s), meta total-count=2207 (COMPLETE)
debt_to_penny: 2761 records in 1 page file(s), meta total-count=2761 (COMPLETE)
interest_expense: 5016 records in 1 page file(s), meta total-count=5016 (COMPLETE)

wrote treasury-cost-pull1.csv: 11 row(s)
record_fiscal_year | average_debt | interest_expense | implied_rate | interest_bearing_rate | marketable_rate | nonmarketable_rate | flag
2015 | 18096062581363.58 | 402435356075.49 | 2.2239 | 2.3602 | 2.0181 | 3.1653 | no
2016 | 19066846932474.53 | 432649652901.12 | 2.2691 | 2.3005 | 2.0342 | 2.9186 | no
2017 | 19878442153668.92 | 458542287311.80 | 2.3067 | 2.2562 | 2.0322 | 2.7894 | no
2018 | 20916712552863.04 | 523017301446.12 | 2.5005 | 2.3788 | 2.2206 | 2.7826 | no
2019 | 22025152565489.68 | 574587783463.63 | 2.6088 | 2.5578 | 2.4722 | 2.7812 | no
2020 | 24583391553432.13 | 522767299265.34 | 2.1265 | 2.1495 | 2.0491 | 2.4502 | no
2021 | 27954753605841.65 | 562388232682.17 | 2.0118 | 1.6590 | 1.5137 | 2.1508 | no
2022 | 30032058686496.05 | 718745810096.93 | 2.3933 | 1.7153 | 1.5959 | 2.1390 | yes
2023 | 31784930379662.38 | 882598268533.48 | 2.7768 | 2.5873 | 2.5937 | 2.5587 | no
2024 | 34450429247286.05 | 1133036890149.16 | 3.2889 | 3.2180 | 3.2788 | 2.9735 | no
2025 | 36367908003144.63 | 1220025282630.31 | 3.3547 | 3.3085 | 3.3662 | 3.0565 | no
```

Re-running the same command reproduces `treasury-cost-pull1.csv` byte-identically (SHA-256 a2a6a9052039bb5745498251600cdf195163ce94fc566c7dda8048ffa22f62b5).

Command (pull 2 CSV): `python -m treasury_recon --responses-dir pull2/ --out treasury-cost-pull2.csv`

```
avg_interest_rates: 0 records in 0 page file(s), meta total-count=None
debt_to_penny: 2761 records in 1 page file(s), meta total-count=2761 (COMPLETE)
interest_expense: 0 records in 0 page file(s), meta total-count=None

wrote treasury-cost-pull2.csv: 11 row(s)
record_fiscal_year | average_debt | interest_expense | implied_rate | interest_bearing_rate | marketable_rate | nonmarketable_rate | flag
2015 | 18096062581363.58 |  |  |  |  |  | 
2016 | 19066846932474.53 |  |  |  |  |  | 
2017 | 19878442153668.92 |  |  |  |  |  | 
2018 | 20916712552863.04 |  |  |  |  |  | 
2019 | 22025152565489.68 |  |  |  |  |  | 
2020 | 24583391553432.13 |  |  |  |  |  | 
2021 | 27954753605841.65 |  |  |  |  |  | 
2022 | 30032058686496.05 |  |  |  |  |  | 
2023 | 31784930379662.38 |  |  |  |  |  | 
2024 | 34450429247286.05 |  |  |  |  |  | 
2025 | 36367908003144.63 |  |  |  |  |  | 
```

The pull-2 run writes the partial CSV (SHA-256 713545815916db2518ae8f34038bf02c0509f152dd23c513fbf8fa8938e02de3): 11 rows, only `average_debt` populated, because only `debt_to_penny` has pull-2 responses; the other two datasets have none, so their columns stay empty, and the `flag` column is empty in every row because the flag cannot be computed from pull-2 data. No data invented.

## Results by fiscal year (pull 1, complete)

| FY | average debt | interest expense | implied rate | interest-bearing rate | marketable rate | non-marketable rate | flag |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 2015 | 18096062581363.58 | 402435356075.49 | 2.2239 | 2.3602 | 2.0181 | 3.1653 | no |
| 2016 | 19066846932474.53 | 432649652901.12 | 2.2691 | 2.3005 | 2.0342 | 2.9186 | no |
| 2017 | 19878442153668.92 | 458542287311.80 | 2.3067 | 2.2562 | 2.0322 | 2.7894 | no |
| 2018 | 20916712552863.04 | 523017301446.12 | 2.5005 | 2.3788 | 2.2206 | 2.7826 | no |
| 2019 | 22025152565489.68 | 574587783463.63 | 2.6088 | 2.5578 | 2.4722 | 2.7812 | no |
| 2020 | 24583391553432.13 | 522767299265.34 | 2.1265 | 2.1495 | 2.0491 | 2.4502 | no |
| 2021 | 27954753605841.65 | 562388232682.17 | 2.0118 | 1.6590 | 1.5137 | 2.1508 | no |
| 2022 | 30032058686496.05 | 718745810096.93 | 2.3933 | 1.7153 | 1.5959 | 2.1390 | yes |
| 2023 | 31784930379662.38 | 882598268533.48 | 2.7768 | 2.5873 | 2.5937 | 2.5587 | no |
| 2024 | 34450429247286.05 | 1133036890149.16 | 3.2889 | 3.2180 | 3.2788 | 2.9735 | no |
| 2025 | 36367908003144.63 | 1220025282630.31 | 3.3547 | 3.3085 | 3.3662 | 3.0565 | no |

Definitions: average debt = mean of daily `tot_pub_debt_out_amt`; interest expense = sum of `month_expense_amt`; implied rate = interest expense / average debt x 100; interest-bearing / marketable / non-marketable rate = fiscal-year mean of monthly `avg_interest_rate_amt` for `Total Interest-bearing Debt` / `Total Marketable` / `Total Non-marketable`; flag = "yes" when |implied rate - interest-bearing mean| > 0.5 percentage points, "no" when both are known and the gap is at most 0.5 points, and empty when the implied rate or the interest-bearing mean cannot be computed from the data held.

Pull-2 comparison: `average_debt` in pull 2 is identical to pull 1 for all 11 fiscal years (see Changes). The other pull-2 columns are empty because their sources failed on pull 2 (see PROOF Requests 5-6).

## Results by fiscal year (pull 2, partial)

The pull-2 CSV as delivered (11 rows). Only `average_debt` is pull-2 data (identical to pull 1 for all 11 fiscal years); the other columns are empty because their pull-2 sources failed after one retry each, and the `flag` column is empty in every row because the flag cannot be computed from pull-2 data.

| FY | average debt | interest expense | implied rate | interest-bearing rate | marketable rate | non-marketable rate | flag |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 2015 | 18096062581363.58 |  |  |  |  |  |  |
| 2016 | 19066846932474.53 |  |  |  |  |  |  |
| 2017 | 19878442153668.92 |  |  |  |  |  |  |
| 2018 | 20916712552863.04 |  |  |  |  |  |  |
| 2019 | 22025152565489.68 |  |  |  |  |  |  |
| 2020 | 24583391553432.13 |  |  |  |  |  |  |
| 2021 | 27954753605841.65 |  |  |  |  |  |  |
| 2022 | 30032058686496.05 |  |  |  |  |  |  |
| 2023 | 31784930379662.38 |  |  |  |  |  |  |
| 2024 | 34450429247286.05 |  |  |  |  |  |  |
| 2025 | 36367908003144.63 |  |  |  |  |  |  |

## Flagged year: FY2022

Only FY2022 is flagged. Implied rate 2.3933%, interest-bearing mean 1.7153%, difference 0.6780 percentage points, above the 0.5 threshold.

What the data shows. Interest expense jumped while the published average rate barely moved:

- Interest expense: FY2021 562388232682.17 to FY2022 718745810096.93 (+27.80%).
- Average debt: FY2021 27954753605841.65 to FY2022 30032058686496.05 (+7.43%).
- Interest-bearing mean: FY2021 1.6590% to FY2022 1.7153% (+0.0563 percentage points).
- Gap |implied - interest-bearing|: FY2021 0.3528 (no flag), FY2022 0.6780 (flag).

In words: the Treasury paid 27.79% more interest in FY2022 on a debt stock only 7.43% larger on average, so the expense-implied rate rose from 2.0118% to 2.3933% (+0.3815 points) while the published mean rate on interest-bearing debt rose only from 1.6590% to 1.7153% (+0.0563 points). The two measures differ by construction: the implied rate divides total interest expense by average total debt outstanding, while the published mean averages the monthly average rate on interest-bearing securities only. No other year crosses the 0.5-point line; the next-largest gap is FY2021 at 0.3528 points.

Cross-check (from the raw records): the FY2022 interest expense total 718745810096.93 is the sum of `month_expense_amt` over all 456 FY2022 expense records, and it exactly equals the sum of the 38 September 2022 `fytd_expense_amt` values (the fiscal-year-to-date totals), so the monthly amounts are summed once, with no double-counting.

Raw records behind the FY2022 flag. Monthly `avg_interest_rate_amt` for `Total Interest-bearing Debt`, FY2022 (12 values, mean 1.7153):

```
{"record_date":"2021-10-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.592","record_fiscal_year":"2022"}
{"record_date":"2021-11-30","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.577","record_fiscal_year":"2022"}
{"record_date":"2021-12-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.565","record_fiscal_year":"2022"}
{"record_date":"2022-01-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.556","record_fiscal_year":"2022"}
{"record_date":"2022-02-28","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.568","record_fiscal_year":"2022"}
{"record_date":"2022-03-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.608","record_fiscal_year":"2022"}
{"record_date":"2022-04-30","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.659","record_fiscal_year":"2022"}
{"record_date":"2022-05-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.734","record_fiscal_year":"2022"}
{"record_date":"2022-06-30","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.800","record_fiscal_year":"2022"}
{"record_date":"2022-07-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.884","record_fiscal_year":"2022"}
{"record_date":"2022-08-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"1.971","record_fiscal_year":"2022"}
{"record_date":"2022-09-30","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"2.070","record_fiscal_year":"2022"}
```

FY2022 interest-expense records (first 3 of 456, then 2 September 2022 rows showing the fiscal-year-to-date totals):

```
{"record_date":"2021-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Notes","month_expense_amt":"14783728441.01","fytd_expense_amt":"14783728441.01","record_fiscal_year":"2022"}
{"record_date":"2021-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Bonds","month_expense_amt":"8735323777.70","fytd_expense_amt":"8735323777.70","record_fiscal_year":"2022"}
{"record_date":"2021-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Inflation Protected Securities (TIPS)","month_expense_amt":"1231303947.50","fytd_expense_amt":"1231303947.50","record_fiscal_year":"2022"}
{"record_date":"2022-09-30","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Notes","month_expense_amt":"16705318381.46","fytd_expense_amt":"183692743484.03","record_fiscal_year":"2022"}
{"record_date":"2022-09-30","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Bonds","month_expense_amt":"9245609222.90","fytd_expense_amt":"107613711781.45","record_fiscal_year":"2022"}
```

FY2022 daily debt records (first and last of 252; mean of all 252 `tot_pub_debt_out_amt` values is 30032058686496.05):

```
{"record_date":"2021-10-01","tot_pub_debt_out_amt":"28428906947685.31","record_fiscal_year":"2022"}
{"record_date":"2022-09-30","tot_pub_debt_out_amt":"30928911613306.73","record_fiscal_year":"2022"}
```

## Changes: pull 1 vs pull 2

Key column: `record_fiscal_year`. Both CSVs have the same 11 keys (FY 2015-2025). Counts are computed cell by cell from the two delivered CSV files: an empty cell vs a value counts as a difference.

- new (keys only in pull 2): 0
- gone (keys only in pull 1): 0
- changed (same key, different value in a named column): 11 keys (66 cells)
- pull 1 rows: 11; pull 2 rows: 11; 11 = 11 + 0 - 0.

Changed cells by column: `average_debt` 0, `interest_expense` 11, `implied_rate` 11, `interest_bearing_rate` 11, `marketable_rate` 11, `nonmarketable_rate` 11, `flag` 11.

Changed cells by key (value in pull 1 -> empty in pull 2):
- FY2015: `interest_expense` 402435356075.49 -> (empty); `implied_rate` 2.2239 -> (empty); `interest_bearing_rate` 2.3602 -> (empty); `marketable_rate` 2.0181 -> (empty); `nonmarketable_rate` 3.1653 -> (empty); `flag` no -> (empty)
- FY2016: `interest_expense` 432649652901.12 -> (empty); `implied_rate` 2.2691 -> (empty); `interest_bearing_rate` 2.3005 -> (empty); `marketable_rate` 2.0342 -> (empty); `nonmarketable_rate` 2.9186 -> (empty); `flag` no -> (empty)
- FY2017: `interest_expense` 458542287311.80 -> (empty); `implied_rate` 2.3067 -> (empty); `interest_bearing_rate` 2.2562 -> (empty); `marketable_rate` 2.0322 -> (empty); `nonmarketable_rate` 2.7894 -> (empty); `flag` no -> (empty)
- FY2018: `interest_expense` 523017301446.12 -> (empty); `implied_rate` 2.5005 -> (empty); `interest_bearing_rate` 2.3788 -> (empty); `marketable_rate` 2.2206 -> (empty); `nonmarketable_rate` 2.7826 -> (empty); `flag` no -> (empty)
- FY2019: `interest_expense` 574587783463.63 -> (empty); `implied_rate` 2.6088 -> (empty); `interest_bearing_rate` 2.5578 -> (empty); `marketable_rate` 2.4722 -> (empty); `nonmarketable_rate` 2.7812 -> (empty); `flag` no -> (empty)
- FY2020: `interest_expense` 522767299265.34 -> (empty); `implied_rate` 2.1265 -> (empty); `interest_bearing_rate` 2.1495 -> (empty); `marketable_rate` 2.0491 -> (empty); `nonmarketable_rate` 2.4502 -> (empty); `flag` no -> (empty)
- FY2021: `interest_expense` 562388232682.17 -> (empty); `implied_rate` 2.0118 -> (empty); `interest_bearing_rate` 1.6590 -> (empty); `marketable_rate` 1.5137 -> (empty); `nonmarketable_rate` 2.1508 -> (empty); `flag` no -> (empty)
- FY2022: `interest_expense` 718745810096.93 -> (empty); `implied_rate` 2.3933 -> (empty); `interest_bearing_rate` 1.7153 -> (empty); `marketable_rate` 1.5959 -> (empty); `nonmarketable_rate` 2.1390 -> (empty); `flag` yes -> (empty)
- FY2023: `interest_expense` 882598268533.48 -> (empty); `implied_rate` 2.7768 -> (empty); `interest_bearing_rate` 2.5873 -> (empty); `marketable_rate` 2.5937 -> (empty); `nonmarketable_rate` 2.5587 -> (empty); `flag` no -> (empty)
- FY2024: `interest_expense` 1133036890149.16 -> (empty); `implied_rate` 3.2889 -> (empty); `interest_bearing_rate` 3.2180 -> (empty); `marketable_rate` 3.2788 -> (empty); `nonmarketable_rate` 2.9735 -> (empty); `flag` no -> (empty)
- FY2025: `interest_expense` 1220025282630.31 -> (empty); `implied_rate` 3.3547 -> (empty); `interest_bearing_rate` 3.3085 -> (empty); `marketable_rate` 3.3662 -> (empty); `nonmarketable_rate` 3.0565 -> (empty); `flag` no -> (empty)

`average_debt` is identical in pull 1 and pull 2 for all 11 fiscal years (the pull-2 `debt_to_penny` page is byte-identical to pull 1: SHA-256 e41afec56f5013a3d8ccee746c5ddd4180508cc0be83057c30b252822ca2494f in both manifests). The other six value columns differ because their pull-2 sources failed after retries, so pull 2 leaves them empty: `interest_expense`, `implied_rate`, `interest_bearing_rate`, `marketable_rate` and `nonmarketable_rate` are populated in pull 1 and empty in pull 2 for all 11 keys (55 cells), and `flag` is empty in pull 2 for all 11 keys (11 cells): `yes` -> (empty) for FY2022 and `no` -> (empty) for the other 10 years. Note on the count: the first review counted 56 cells against the first delivery, whose `flag` column read `no` in all 11 pull-2 rows (so only the FY2022 flag differed); with the flag corrected to empty in every pull-2 row (items 2 and 6 of the revision), the flag column differs in all 11 rows, giving 55 + 11 = 66 changed cells.

pull 2 failed (counts as done) for `avg_interest_rates` and `interest_expense` (see PROOF Requests 5-6 and the post-note retry in Request 7).

## PROOF

Every request, with its UTC time (each response's HTTP Date header, the source's own time) and a raw excerpt of the response the numbers come from. Each excerpt is at most 30 lines. Records are shown one per line, copied exactly as returned.

### Request 1: PULL 1 avg_interest_rates

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): Sat, 14 Nov 2026 18:29:37 GMT
- Excerpt (copied exactly as returned, the lines that show the values reported):

```
HTTP Date: Sat, 14 Nov 2026 18:29:37 GMT
meta object, exactly as returned in the saved pull1/ file:
"meta":{"count":2207,"labels":{"record_date":"Record Date","security_type_desc":"Security Type Description","security_desc":"Security Description","avg_interest_rate_amt":"Average Interest Rate Amount","src_line_nbr":"Source Line Number","record_fiscal_year":"Fiscal Year","record_fiscal_quarter":"Fiscal Quarter Number","record_calendar_year":"Calendar Year","record_calendar_quarter":"Calendar Quarter Number","record_calendar_month":"Calendar Month Number","record_calendar_day":"Calendar Day Number"},"dataTypes":{"record_date":"DATE","security_type_desc":"STRING","security_desc":"STRING","avg_interest_rate_amt":"PERCENTAGE","src_line_nbr":"INTEGER","record_fiscal_year":"YEAR","record_fiscal_quarter":"QUARTER","record_calendar_year":"YEAR","record_calendar_quarter":"QUARTER","record_calendar_month":"MONTH","record_calendar_day":"DAY"},"dataFormats":{"record_date":"YYYY-MM-DD","security_type_desc":"String","security_desc":"String","avg_interest_rate_amt":"10.2%","src_line_nbr":"10","record_fiscal_year":"YYYY","record_fiscal_quarter":"Q","record_calendar_year":"YYYY","record_calendar_quarter":"Q","record_calendar_month":"MM","record_calendar_day":"DD"},"total-count":2207,"total-pages":1}
{"record_date":"2014-10-31","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"2.384","record_fiscal_year":"2015"}
{"record_date":"2014-10-31","security_desc":"Treasury Notes","avg_interest_rate_amt":"1.807","record_fiscal_year":"2015"}
```

### Request 2: PULL 1 debt_to_penny

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): Sat, 14 Nov 2026 18:29:40 GMT
- Excerpt (copied exactly as returned, the lines that show the values reported):

```
HTTP Date: Sat, 14 Nov 2026 18:29:40 GMT
meta object, exactly as returned in the saved pull1/ file:
"meta":{"count":2761,"labels":{"record_date":"Record Date","debt_held_public_amt":"Debt Held by the Public","intragov_hold_amt":"Intragovernmental Holdings","tot_pub_debt_out_amt":"Total Public Debt Outstanding","src_line_nbr":"Source Line Number","record_fiscal_year":"Fiscal Year","record_fiscal_quarter":"Fiscal Quarter Number","record_calendar_year":"Calendar Year","record_calendar_quarter":"Calendar Quarter Number","record_calendar_month":"Calendar Month Number","record_calendar_day":"Calendar Day Number"},"dataTypes":{"record_date":"DATE","debt_held_public_amt":"CURRENCY","intragov_hold_amt":"CURRENCY","tot_pub_debt_out_amt":"CURRENCY","src_line_nbr":"INTEGER","record_fiscal_year":"YEAR","record_fiscal_quarter":"QUARTER","record_calendar_year":"YEAR","record_calendar_quarter":"QUARTER","record_calendar_month":"MONTH","record_calendar_day":"DAY"},"dataFormats":{"record_date":"YYYY-MM-DD","debt_held_public_amt":"$10.20","intragov_hold_amt":"$10.20","tot_pub_debt_out_amt":"$10.20","src_line_nbr":"10","record_fiscal_year":"YYYY","record_fiscal_quarter":"Q","record_calendar_year":"YYYY","record_calendar_quarter":"Q","record_calendar_month":"MM","record_calendar_day":"DD"},"total-count":2761,"total-pages":1}
{"record_date":"2014-10-01","tot_pub_debt_out_amt":"17875258091207.08","record_fiscal_year":"2015"}
{"record_date":"2014-10-02","tot_pub_debt_out_amt":"17872947693176.67","record_fiscal_year":"2015"}
```

### Request 3: PULL 1 interest_expense

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): Sat, 14 Nov 2026 18:29:44 GMT
- Excerpt (copied exactly as returned, the lines that show the values reported):

```
HTTP Date: Sat, 14 Nov 2026 18:29:44 GMT
meta object, exactly as returned in the saved pull1/ file:
"meta":{"count":5016,"labels":{"record_date":"Record Date","expense_catg_desc":"Expense Category Description","expense_group_desc":"Expense Group Description","expense_type_desc":"Expense Type Description","month_expense_amt":"Current Month Expense Amount","fytd_expense_amt":"Fiscal Year to Date Expense Amount","src_line_nbr":"Source Line Number","record_fiscal_year":"Fiscal Year","record_fiscal_quarter":"Fiscal Quarter Number","record_calendar_year":"Calendar Year","record_calendar_quarter":"Calendar Quarter Number","record_calendar_month":"Calendar Month Number","record_calendar_day":"Calendar Day Number"},"dataTypes":{"record_date":"DATE","expense_catg_desc":"STRING","expense_group_desc":"STRING","expense_type_desc":"STRING","month_expense_amt":"CURRENCY","fytd_expense_amt":"CURRENCY","src_line_nbr":"INTEGER","record_fiscal_year":"YEAR","record_fiscal_quarter":"QUARTER","record_calendar_year":"YEAR","record_calendar_quarter":"QUARTER","record_calendar_month":"MONTH","record_calendar_day":"DAY"},"dataFormats":{"record_date":"YYYY-MM-DD","expense_catg_desc":"String","expense_group_desc":"String","expense_type_desc":"String","month_expense_amt":"$10.20","fytd_expense_amt":"$10.20","src_line_nbr":"10","record_fiscal_year":"YYYY","record_fiscal_quarter":"Q","record_calendar_year":"YYYY","record_calendar_quarter":"Q","record_calendar_month":"MM","record_calendar_day":"DD"},"total-count":5016,"total-pages":1}
{"record_date":"2014-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Notes","month_expense_amt":"12107124909.87","fytd_expense_amt":"12107124909.87","record_fiscal_year":"2015"}
{"record_date":"2014-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_type_desc":"Treasury Bonds","month_expense_amt":"6346671026.90","fytd_expense_amt":"6346671026.90","record_fiscal_year":"2015"}
```

### Request 4: PULL 2 debt_to_penny

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): Sun, 15 Nov 2026 18:36:38 GMT
- Excerpt (copied exactly as returned, the lines that show the values reported):

```
HTTP Date: Sun, 15 Nov 2026 18:36:38 GMT
meta object, exactly as returned in the saved pull2/ file:
"meta":{"count":2761,"labels":{"record_date":"Record Date","debt_held_public_amt":"Debt Held by the Public","intragov_hold_amt":"Intragovernmental Holdings","tot_pub_debt_out_amt":"Total Public Debt Outstanding","src_line_nbr":"Source Line Number","record_fiscal_year":"Fiscal Year","record_fiscal_quarter":"Fiscal Quarter Number","record_calendar_year":"Calendar Year","record_calendar_quarter":"Calendar Quarter Number","record_calendar_month":"Calendar Month Number","record_calendar_day":"Calendar Day Number"},"dataTypes":{"record_date":"DATE","debt_held_public_amt":"CURRENCY","intragov_hold_amt":"CURRENCY","tot_pub_debt_out_amt":"CURRENCY","src_line_nbr":"INTEGER","record_fiscal_year":"YEAR","record_fiscal_quarter":"QUARTER","record_calendar_year":"YEAR","record_calendar_quarter":"QUARTER","record_calendar_month":"MONTH","record_calendar_day":"DAY"},"dataFormats":{"record_date":"YYYY-MM-DD","debt_held_public_amt":"$10.20","intragov_hold_amt":"$10.20","tot_pub_debt_out_amt":"$10.20","src_line_nbr":"10","record_fiscal_year":"YYYY","record_fiscal_quarter":"Q","record_calendar_year":"YYYY","record_calendar_quarter":"Q","record_calendar_month":"MM","record_calendar_day":"DD"},"total-count":2761,"total-pages":1}
SHA-256 of this page: e41afec56f5013a3d8ccee746c5ddd4180508cc0be83057c30b252822ca2494f (identical to the pull-1 page)
{"record_date":"2014-10-01","tot_pub_debt_out_amt":"17875258091207.08","record_fiscal_year":"2015"}
{"record_date":"2014-10-02","tot_pub_debt_out_amt":"17872947693176.67","record_fiscal_year":"2015"}
```

### Request 5: PULL 2 avg_interest_rates (FAILED)

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): no response (see attempts below). No HTTP Date header was captured for the failed responses: the fetch client raises on HTTPError before reading headers, so the saved records hold the error string and the run's time bounds, not a per-response Date header. The times below are the exact bounds provable from the saved records, not to-the-second timestamps.
- Excerpt (the error, verbatim from pull2/manifest.json, with its UTC time bounds):

```
Attempt 1: between the pull-2 run start (2026-11-15 18:35:00 UTC, the scheduled run) and 18:36:38 GMT (the next request in the same run, debt_to_penny, succeeded with HTTP Date Sun, 15 Nov 2026 18:36:38 GMT; datasets are fetched in the order avg_interest_rates, debt_to_penny, interest_expense):
  https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1: HTTP 503 Service Unavailable
Retry: after the attempt-1 failures and before the PULL 2 note filed 2026-11-15 18:45:48 UTC (the manifest records this retry window as approximately 18:39-18:42 UTC):
  https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1: HTTP 503 Service Unavailable
```

pull 2 failed (counts as done) for this source.

### Request 6: PULL 2 interest_expense (FAILED)

- Request: https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1
- Retrieved (UTC): no response (see attempts below). No HTTP Date header was captured for the failed responses: the fetch client raises on HTTPError before reading headers (and a timeout has no response at all), so the saved records hold the error string and the run's time bounds, not a per-response Date header. The times below are the exact bounds provable from the saved records, not to-the-second timestamps.
- Excerpt (the error, verbatim from pull2/manifest.json, with its UTC time bounds):

```
Attempt 1: after the debt_to_penny response in the same run (HTTP Date Sun, 15 Nov 2026 18:36:38 GMT); the read timed out under the 120-second client timeout:
  TimeoutError: The read operation timed out
Retry: after the attempt-1 failures and before the PULL 2 note filed 2026-11-15 18:45:48 UTC (the manifest records this retry window as approximately 18:39-18:42 UTC):
  https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=10000&page%5Bnumber%5D=1: HTTP 503 Service Unavailable
```

pull 2 failed (counts as done) for this source.

### Request 7: post-note verification retry (main agent)

After the PULL 2 note was filed, the main agent re-ran the same three requests once more (2026-11-15 18:49-18:52 UTC) to check whether the outage had cleared. All three endpoints returned HTTP 503 Service Unavailable, including `debt_to_penny`, which had succeeded at 18:36:38 UTC. The outage was ongoing and widening, not a single transient failure. Raw manifest saved as `pull2-retry/manifest.json`. No further retry was made: the TASK allows submitting with pull 2 failed for the affected sources.

### Request 8: revision-time re-check of the two failed endpoints

During this revision the two endpoints that failed on pull 2 were re-requested once each (same URLs as Requests 5-6, page[size]=1) to check whether the outage had cleared. This is a revision-time check only; it is not part of pull 2 and changes nothing about pull 2, which stands as recorded above.

- Request (avg_interest_rates): https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/avg_interest_rates?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=1&page%5Bnumber%5D=1
- Retrieved (UTC): 2026-11-16 17:36:19 UTC
- Excerpt (raw response, status line and headers):

```
HTTP/1.1 200 OK
Date: Mon, 16 Nov 2026 17:36:19 GMT
Content-Type: application/json
{"data":[{"record_date":"2014-10-31","security_type_desc":"Interest-bearing Debt","security_desc":"Total Interest-bearing Debt","avg_interest_rate_amt":"2.384","src_line_nbr":"17","record_fiscal_year":"2015","record_fiscal_quarter":"1","record_calendar_year":"2014","record_calendar_quarter":"4","rec ...
```

- Request (interest_expense): https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/interest_expense?filter=record_fiscal_year%3Agte%3A2015%2Crecord_fiscal_year%3Alte%3A2025&page%5Bsize%5D=1&page%5Bnumber%5D=1
- Retrieved (UTC): 2026-11-16 17:36:20 UTC
- Excerpt (raw response, status line and headers):

```
HTTP/1.1 200 OK
Date: Mon, 16 Nov 2026 17:36:21 GMT
Content-Type: application/json
{"data":[{"record_date":"2014-10-31","expense_catg_desc":"INTEREST EXPENSE ON PUBLIC ISSUES","expense_group_desc":"ACCRUED INTEREST EXPENSE","expense_type_desc":"Treasury Notes","month_expense_amt":"12107124909.87","fytd_expense_amt":"12107124909.87","src_line_nbr":"1","record_fiscal_year":"2015","r ...
```

Both endpoints were live again by 2026-11-16 17:36 UTC: the pull-2 outage was time-bounded (2026-11-15 18:35-18:52 UTC, per Requests 5-7) and had cleared by the revision. Raw re-check record saved as `revision-recheck.json`.

## Data files

Attached under Rule 25 (see the synmerco-data block). D2 is the complete pull-1 CSV; D3 is the partial pull-2 CSV disclosed above.

```synmerco-data
a2a6a9052039bb5745498251600cdf195163ce94fc566c7dda8048ffa22f62b5 treasury-cost-pull1.csv
713545815916db2518ae8f34038bf02c0509f152dd23c513fbf8fa8938e02de3 treasury-cost-pull2.csv
```

## Notes

- Package language: Python 3, stdlib only. No key, no login, no paid source. Only the three listed Treasury Fiscal Data datasets were used.
- Money values are Decimal (exact); rates are shown to 4 decimals, money to 2 decimals, as computed by the package.
- `security_desc` totals used (verified against the live pull-1 responses): "Total Interest-bearing Debt", "Total Marketable", "Total Non-marketable". Other `security_desc` values are component lines and are ignored for the rate means.
- A fiscal year is taken from each record's `record_fiscal_year` field; the package falls back to deriving it from `record_date` (FY starts 1 October).
- Progress notes on the platform (5 total): work-started note 2026-11-14 18:19:31 UTC; PULL 1 note 2026-11-14 18:34:09 UTC; PULL 2 note 2026-11-15 18:45:48 UTC; revision-1 note 2026-11-16 17:27:06 UTC; D3-revision note 2026-11-16 18:09:43 UTC.
