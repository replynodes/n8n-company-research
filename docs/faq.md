# FAQ

## What is this?
An n8n workflow template that turns a list of company domains into structured company profiles using the companies' own websites and your LLM.

## Is it the same as company enrichment?
No. Enrichment tools sell headcount, funding, revenue and emails from databases. This template reads public websites and returns what the site states, with evidence URLs. It never guesses.

## Which data does it return?
Name, description, industry, products, target customers, free-trial flag, pricing summary, integrations, logo, brand colors, key page links, social links, evidence URLs and a status. See [output.schema.json](output.schema.json).

## Do I need an API key for the data sources?
No. The Markdown, brand and logo endpoints work without a key within public limits. You only need an LLM credential.

## What are the limits?
About 20 requests per minute per IP on the Markdown and brand endpoints. The default 12-second pause between domains stays below that. Rows hit by a limit get `rate_limited`.

## Why is a row `needs_review`?
The page returned too little text (often a JavaScript-only site) or the LLM could not fill a main field. Check `error` and `evidence_urls`.

## Why is a row `blocked`?
The site refused the Markdown fetch. Try the domain manually; the workflow does not bypass blocks.

## Can I use Claude, Gemini or a local model?
Yes. Swap the LLM node for any n8n chat model.

## Can I run it on a schedule or from a Sheet?
Yes. Replace the Manual Trigger with a Schedule Trigger and the Domains node with a Google Sheets read.

## What does the X-Source header do?
It lets the endpoint operator count template usage. It is set in the Settings node and can be changed or removed.

## Where do I report problems?
Open a GitHub issue with the domain, the `status` and `error` values.
