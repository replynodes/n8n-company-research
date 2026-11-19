# ICP company list builder (n8n + Google Sheets)

Describe the companies you want (a keyword, countries, and a size range), run the workflow, and it appends matching companies to a Google Sheet with name, domain, description, industry, HQ, employee range and funding. Companies already in the sheet are skipped.

## What you need

- n8n (tested on 2.35.7) and a Google account.
- A ReplyNodes API key. New accounts get 500 free credits: https://replynodes.com/auth.md

## Cost

**1 credit per page of up to 10 companies.** Empty pages and errors are free. `maxPages` caps the spend: the default of 5 costs at most 5 credits per run (50 companies). See https://replynodes.com for current pricing.

## Setup

1. **Create the sheet.** Paste this header row into row 1:

   ```
   domain	status	source	name	description	industries	hq_country	hq_region	employees_min	employees_max	funding_total_usd	latest_round_type	latest_round_date	data_loaded_at	checked_at	error	query	page
   ```

2. **Import** [`workflow.json`](workflow.json): Workflows, Import from File.
3. **Google Sheets:** open **Read existing rows** and **Add to sheet**, connect your Google account, and pick the same spreadsheet and tab in both.
4. **API key:** create a **Header Auth** credential named `ReplyNodes API` with header name `Authorization` and value `Bearer <your key>`. Select it in **Search page**.
5. Open **Settings**, describe your target, and click **Execute workflow**.

## Settings

| setting | meaning |
|---|---|
| `keyword` | one term or a quoted phrase, for example `carbon accounting software` or `"customer support"` |
| `countries` | ISO-2 codes separated by commas, for example `US,GB`. Empty means any country |
| `employeesMin`, `employeesMax` | company size range. `null` means no limit |
| `industries` | optional, names separated by `\|`. Empty means any |
| `maxPages` | pages of 10 to fetch, default 5, capped at 10 |

## How search behaves

- Search matches your keyword against company records and ranks by match and prominence. It is **not** a natural-language question: "top 10 fintech startups in Berlin" will not work. Use a plain keyword plus filters.
- Avoid `OR` and `-exclusion` in the keyword. Those queries often exceed the API's 5-second limit and time out.
- A search returns at most 1,000 results in total. Narrow your keyword if you hit the end of the list.
- Pages are fetched one at a time with a pause between them. A 429, 503 or 504 is retried once; if that also fails, the companies already collected are still added to your sheet, and **Run summary** says why the run stopped.

## What you get back

**Run summary** (a node output you can open in n8n) shows pages fetched, companies added, duplicates skipped and the stop reason (`no more results`, `reached max_pages`, or the error). Each added row carries the `query` and `page` it came from, `status = found`, and `source = replynodes_db`.

## Know the limits of the data

The database is built from third-party datasets (`data_loaded_at` shows when it was last loaded). Spot-check a sample before you use a list for outreach.

- Some records are old, and some fields are empty. Funding is present on roughly 6 in 10 records and a description on about 8 in 10.
- Keyword results can include loosely related companies. Review the `description` column.
- Industry labels come from the source datasets and can look odd.

## Tested

Run end to end on n8n 2.35.7: a 3-page run (no duplicates; a domain already in the sheet was skipped), a run capped by `maxPages`, a single-page keyword, an empty result, a missing keyword (clear error), and a run where page 2 fails twice (page 1 rows are kept and the reason is reported). The API responses used in that run were captured from the live service. The Google Sheets nodes use n8n's built-in node and were not run against a real spreadsheet in that test, so tell us if the sheet step misbehaves: [open an issue](../../issues).

## Attribution

Requests carry an `X-Source: n8n-company-research/icp-list-builder` header so ReplyNodes can count template usage. Only your search settings are sent. Change or remove it in the **Settings** node and the **Search page** node.
