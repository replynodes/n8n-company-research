# n8n Company Research: company profiles from a domain

An import-and-run n8n workflow. Give it a list of company domains and it returns a structured profile built from each company's own website: name, description, industry, products, target customers, brand assets, key links and pricing signals, with the pages used as evidence.

No data-provider API key is needed. The only credential you add is the LLM you already use in n8n.

> **Status: work in progress.** `workflow.json` is not published yet. This README describes the v1 target. Star or watch the repo to be notified when it lands, and delete this note once it has.

## What you get

One row per domain. Fields are grouped so a Google Sheet stays readable.

| Group | Fields |
|---|---|
| Identity | `domain`, `company_name`, `description`, `industry`, `products`, `target_customers` |
| Brand | `logo_url`, `brand_colors` |
| Site links | `pricing_url`, `docs_url`, `changelog_url`, `blog_url`, `careers_url`, `contact_url`, `social_links`, `has_llms_txt` |
| Commercial | `has_free_trial`, `pricing_summary`, `integrations` |
| Evidence and status | `source_url`, `evidence_url`, `status` |

- Link and brand fields are extracted from page and brand data where found, and left empty otherwise.
- Descriptive fields (`description`, `industry`, `products`, `target_customers`, `pricing_summary`, `integrations`) are summarized by your LLM, and each is meant to carry an `evidence_url` so you can check it.
- `industry` is constrained to a short fixed list so you can filter and sort on it.

### `status` values

`ok`, `needs_review` (a main field is missing), `blocked` (the site refused the fetch), `rate_limited`, `failed`.

A bad domain never stops the run: it gets a status and the workflow moves on to the next one.

## What this is not

This workflow reads public websites. It does not return headcount, funding, revenue or contact emails, because websites rarely state them reliably. If you need those, use a dedicated data provider; this workflow will not guess them.

## Requirements

- n8n (Cloud or self-hosted)
- One LLM credential configured in n8n (OpenAI, Anthropic or any provider the n8n LLM nodes support)

## Quick start

1. Download `workflow.json` from this repo.
2. In n8n, open **Workflows → Import from File** and select it.
3. Select your LLM credential on the LLM node.
4. Run it. It ships with three sample domains so you see output straight away.
5. Replace the sample domains with your own list, or connect a Google Sheet.

## How it works

1. Manual Trigger and a Set node holding the domains.
2. A Code node normalizes each domain (removes scheme, `www`, paths).
3. A loop processes domains in small batches with a short wait between them.
4. HTTP Request fetches the homepage as Markdown from `md.replynodes.com`.
5. HTTP Request fetches the pricing page. A missing page is not an error.
6. HTTP Request fetches brand data from `brand.replynodes.com`.
7. A merge step combines the three sources and trims the Markdown to a sensible length.
8. An LLM node returns structured JSON with evidence URLs.
9. A final Code node combines extracted and LLM fields and sets `status`.

All HTTP nodes continue on error, so one failing domain does not stop the rest.

## Limits

The free endpoints need no API key but have daily limits. When a limit is hit, affected rows are marked `rate_limited`. To raise the limits, register an account by following the guide at https://replynodes.com/auth.md.

## Attribution and privacy

Requests sent by this workflow carry an `X-Source: n8n-company-research` header so ReplyNodes can see how many people use the template. Only the domains you submit are sent to the endpoints. You can remove the header in the HTTP Request nodes.

## Roadmap

- Google Sheets input and output variant
- Listing on the n8n template library
- Optional branch for ReplyNodes API key holders (extra sources and change tracking)

## License

MIT, see [LICENSE](LICENSE).
