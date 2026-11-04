# n8n Company Research: company profile from a domain (no data-provider API key)

An import-and-run **n8n workflow template** for **company research from a website domain**. Give it a list of domains and it returns one structured profile per company, built from the company's own website: name, description, industry, products, target customers, logo, brand colors, key links, pricing signals and the pages used as evidence.

- **One credential only:** the LLM you already use in n8n. No Clearbit, Apollo, Hunter or other data-provider key.
- **Import, run, done:** ships with three sample domains.
- **Honest by design:** every row carries a `status` and evidence URLs. Missing data stays empty instead of being guessed.

> Tested against a local mock of the endpoints on n8n 2.35.7. Real-world feedback is welcome: please [open an issue](../../issues) with the domain and the `status` you got.

## Quick start

1. Download [`workflow.json`](workflow.json).
2. In n8n: **Workflows → Import from File**, select it.
3. Open the node **LLM (replace with your model)**, pick your credential (or swap the node for Anthropic, Gemini, Ollama or any chat model; it is a standard LangChain model input).
4. Open the **Domains** node and replace the sample list (`stripe.com`, `linear.app`, `replynodes.com`) with yours.
5. Click **Execute workflow**. Rows appear on the **Results** node.

Tune pacing and size limits in the **Settings** node (`waitSeconds`, `maxHomepageChars`, ...). Details: [docs/how-it-works.md](docs/how-it-works.md).

## Output fields

One row per domain. Full JSON Schema: [docs/output.schema.json](docs/output.schema.json). Example shape: [examples/sample-output.illustrative.json](examples/sample-output.illustrative.json) (fictional, not real output).

| Group | Fields |
|---|---|
| Identity | `domain`, `company_name`, `description`, `industry`, `products`, `target_customers` |
| Brand | `logo_url`, `logo_resolved`, `brand_colors` |
| Site links | `pricing_url`, `docs_url`, `changelog_url`, `blog_url`, `careers_url`, `contact_url`, `social_links` |
| Commercial | `has_free_trial`, `pricing_summary`, `integrations` |
| Evidence and status | `source_url`, `evidence_urls`, `status`, `error`, `checked_at` |

`industry` is limited to a fixed list of 17 values so you can filter and group on it. Link and brand fields are extracted from the page and brand data where present and left empty otherwise. Descriptive fields come from your LLM, which is told to use only the provided text.

### `status` values

| status | meaning |
|---|---|
| `ok` | profile built |
| `needs_review` | page too short or a main field is missing; check `error` |
| `blocked` | the site refused the fetch (401, 403, 451) |
| `rate_limited` | the free endpoint returned 429; wait or register for higher limits |
| `failed` | invalid domain, HTTP error, or LLM failure; see `error` |

A bad domain never stops the run. It gets a status and the loop moves on.

## What this is not

This is **company research from a website**, not data enrichment. It does not return headcount, funding, revenue or contact emails, because websites rarely state them reliably, and it will not guess them. For those, use a dedicated data provider.

## FAQ

**How do I research a company in n8n without Clearbit or Apollo?**
Fetch the company's homepage and pricing page as Markdown, let an LLM extract a fixed set of fields, and keep the source URLs. This template does exactly that. See [docs/faq.md](docs/faq.md).

**Which API keys do I need?** Only an LLM credential. The Markdown, brand and logo endpoints are keyless within their public limits.

**Does it work with Claude, Gemini or local models?** Yes. Replace the LLM node with any n8n chat model. It uses a Structured Output Parser, so models with reliable JSON output work best.

**Can I use it with Google Sheets or a CRM?** Swap the Domains code node for a Sheets/CRM read node and append the Results to a write node. Keep the Loop and Wait nodes.

**How many domains per run?** The free endpoints allow roughly 20 requests per minute per IP. The default pacing (12 seconds per domain) stays under that. Longer lists work, just slowly.

**Is the data accurate?** It is only as accurate as the company's own website. Check `evidence_urls`, and treat `needs_review` rows manually.

## How it works

Manual Trigger → Settings → Domains (normalize and dedupe) → Loop → fetch homepage Markdown → parse links → fetch pricing page, brand data and logo check → collect → LLM with structured output → build row → wait → Results. Every HTTP node continues on error. Full walkthrough: [docs/how-it-works.md](docs/how-it-works.md).

## Limits

Free endpoints need no key but have public rate limits (about 20 requests/minute/IP for Markdown and brand). When hit, rows get `rate_limited`. To raise limits, register by following https://replynodes.com/auth.md.

## Attribution and privacy

Requests carry an `X-Source: n8n-company-research` header so the endpoint operator, [ReplyNodes](https://replynodes.com), can count template usage. Only the domains you submit are sent. Change or remove it in the **Settings** node (`sourceHeader`).

## Related

- [Free Markdown, brand and logo API](https://github.com/replynodes/free-markdown-brand-logo-api): the endpoints this template calls.

## Roadmap

- Google Sheets input/output variant
- n8n template library listing
- Optional branch for API key holders

## License

MIT, see [LICENSE](LICENSE).
