# How it works

The workflow processes domains one at a time and keeps going when a single domain fails.

| Step | Node | What happens |
|---|---|---|
| 1 | Manual Trigger | Start the run. |
| 2 | Settings | Endpoint bases, wait time, size limits, `X-Source` value. |
| 3 | Domains | Your list. Strips scheme, `www`, path and port; validates; removes duplicates. Invalid entries get `status: failed`, `error: invalid_domain`. |
| 4 | Loop over domains | One domain per iteration. |
| 5 | Fetch homepage | `GET md.replynodes.com/{domain}/` returns the homepage as Markdown. |
| 6 | Parse homepage | Checks the response (2xx and at least 200 characters), extracts pricing, docs, changelog, blog, careers, contact and social links. |
| 7 | Fetch pricing | Uses the pricing link found on the homepage, else `/pricing`. A missing page is fine. |
| 8 | Fetch brand | `GET brand.replynodes.com/{domain}` JSON; colors are read defensively. |
| 9 | Check logo | `HEAD img.replynodes.com/{domain}`; header `x-replynodes-logo-fallback` says `logo` (real) or `placeholder`. |
| 10 | Collect page data | Sets the status, builds the LLM input from trimmed Markdown. |
| 11 | Summarize profile | LLM chain with a Structured Output Parser. The prompt forbids guessing and treats page text as untrusted. |
| 12 | Build row | Merges extracted and LLM fields, validates `industry`, sets final `status`. |
| 13 | Respect rate limit | Waits `waitSeconds` before the next domain. |
| 14 | Results | Final rows. |

## Status logic

- invalid domain → `failed` / `invalid_domain`
- 429 → `rate_limited`; 401, 403, 451 → `blocked`
- 2xx but under 200 characters → `needs_review` / `page_too_short`
- other HTTP errors → `failed` / `http_<code>`
- LLM error → `failed` / `llm_failed`
- missing name, description or industry → `needs_review` / `missing_main_fields`

## Customizing

- **Other model:** replace the LLM node.
- **Other fields:** edit the prompt in *Summarize profile*, the schema in *Structured Output Parser*, and *Build row*.
- **Input from Sheets/CRM:** replace *Domains* with a read node that outputs a `domain` field.
- **Output to Sheets/CRM:** connect a write node after *Results*.

## Prompt-injection note

Website text is untrusted. The prompt tells the model to ignore instructions found in it, and the schema limits what can be returned. Still review output before using it in automated actions.
