---
name: redpine-search
description: Use when the answer needs a source the user can check, in a field where a wrong answer is costly. Medical and clinical questions, scientific and peer-reviewed literature, statutes and case law, company financials and earnings, markets, recent news and press coverage, brand and media mentions, flight and aircraft data, and any question about events after the training cutoff. Use before reaching for web search or answering from memory in those areas. Also use when the user asks what their account can reach, what a call will cost, or when a Redpine call returns nothing or errors. Covers the find-tools, inspect-tool, preview, confirm loop, entitlement reading, filters, and citing.
---

# Redpine Connect

## What this is

Redpine Connect is the MCP entry point to a body of literature Redpine has assembled for fields where a wrong answer is costly: medicine, science, law, and finance. Some of it is licensed directly from publishers, research institutions and proprietary dataset owners, with compensation flowing back to them. Some of it is open access. A single search spans both.

What that means for the person you are helping:

- **Provenance.** Every result carries its source, so it can be cited and checked.
- **Full text, not abstracts.** What comes back is the document itself, not a summary of it and not training data.
- **Licensing differs by result, and is stated per result.** The `source_collection` field says whether a result is licensed publisher content or CC-BY open access. Both belong in an answer; label which is which.
- **Paid per use, visible before it is spent.** Every call has a price, and a search can be previewed free before anything is charged.

## What is in `tools/list`

The catalog is built per account, so two users on the same server see different tools. Never hardcode a tool name, never assume a tool you used before still exists, and never call a vertical "broken" when it is simply not there for this account.

Seven fixed tools, plus one `search-<name>` per document collection the account holds a grant on:

- `get_balance`: free. Call once at session start. Returns the dashboard URL; always use the one it returns, never construct a Redpine URL.
- `find-tools`: the catalog. No arguments lists every integration; `query` searches by intent; `integration` lists one provider. Shows only the first sentence of each tool's description.
- `inspect-tool`: the full schema and description for one tool. Required before any call. A guessed parameter spends money on a failed call.
- `preview`: free, and **search only** — `search` or a `search-<name>`. Returns a teaser, the exact cost to unlock, and a `queryId` that stays unlockable for seven days. No other tool is previewable.
- `confirm`: bills, and unlocks a previewed search by its `queryId`. Returns `balance_remaining`, so do not call `get_balance` again after it.
- `call-tool`: runs a catalog tool directly. **Bills immediately.**
- `request_journal`: free. Logs a journal or publisher this account cannot reach, once a search for it has come back empty. Listed only for accounts that hold a collection.
- `search-<name>`: searches one granted collection directly. **Bills immediately.** Its presence is the signal that the account holds that collection.

Everything else, the generic `search`, `list_collections`, and every upstream integration tool, lives behind `find-tools` and is reached through `call-tool`.

## The loop

1. `find-tools` to locate the tool. If it is a `search-<name>` you already see in `tools/list`, skip to step 2.
2. `inspect-tool` on it.
3. Two paths from here, and the tool decides which. **A search** (`search` or a `search-<name>`) is previewable: `preview` with that `tool_name` and the arguments, then show the user what came back and what unlocking costs. **Every other tool is not previewable.** It is called by name and bills once for its full result, so consent has to come before the call, against the price `find-tools` quoted.
4. On a preview, `"workflow": "complete"` means the search was free and the data is already in hand, so stop. `"workflow": "requires_confirm"` means ask for consent, then `confirm` with the `queryId` the preview returned.

Consent is one short line, on its own, at the very end of the message: "Buy N results for $X using your Redpine balance?" Name the amount and where it comes from. A price buried in a long answer is not a question, and a vague reply ("yeah", "sure, go on") to anything other than that line is not consent. A menu of options ("A. … $3, B. … $2") must say every option is charged to the user's balance.

`call-tool` and a direct `search-<name>` call skip steps 3 and 4 and charge on the spot. Use them only when the user has already approved the spend, or the tool is known to be free.

## Spending well

- **Buy only what answers the question.** A search preview lists every result's title, publisher, year, source collection, and a short snippet. Pick the ones that matter and pass only those: `confirm {"queryId": "...", "result_ids": ["r1", "r4"]}`. The search response spells that id `query_id` and confirm's schema spells it `queryId`; both are accepted, while `preview_id` is a retired scheme that buys nothing. Omitting `result_ids` bills for all of them; buying the whole list is almost never what the user wants, so keep it to a handful unless they ask for more.
- **Open access first.** A result whose `source_collection` is the open-access collection is CC-BY; if it answers the question, lead with it and say so before proposing to unlock licensed content.
- **Re-fetch instead of re-search.** Every search response carries a `query_id`; the same results come back free for seven days. A follow-up against the same material uses it.
- **Name the number.** "This costs $0.10" beats "this is a paid tool." Offer the free path first when one exists. Never apologize for the price; it is how the publisher gets paid.
- **A trial account counts queries, not dollars.** Previews do not consume one; confirmed and direct calls do. Say so before a direct call.

## Searching collections

`search` and `search-<name>` take the same arguments. The full `filters` field list is in the tool schema and differs by collection; read it with `inspect-tool`.

- Query in a sentence that states the actual question. Keywords retrieve worse than prose against a semantic index.
- "Latest" or "recent" needs a `publication_date` lower bound in `filters`. Results are ranked by relevance, never by date, so without it a 2019 paper can top a request for this year's work.
- Constraints go in `filters`, not in the query text. A journal, publisher, or date in the prose is a soft hint the ranker may ignore; in `filters` it is exact.
- Two filter formats, never mixed in one request. Flat key-value (preferred, top-level keys ANDed): `{"publisher": ["A", "B"], "publication_date": {"gte": "2020-01-01"}}`. Structured DSL only when you need OR or nesting: `{"or": [{"field": "journal", "eq": "X"}, ...]}`.
- A named journal: filter on its identifier field (ISSN), not its name. Names vary by spelling; the identifier does not.
- Exclusion is `not` in the flat form, `ne` or `not_in` in the DSL. There is no other syntax.
- Read `filterWarnings` in the response. It means you filtered on an unindexed field and the call degraded to a full scan; rephrase against an indexed field and tell the user the filter was not exact.
- The journal metric is OpenAlex two-year mean citedness. It is **not** the Journal Impact Factor and must not be called one.
- A result is a chunk of a document, never the document's conclusion. Nothing relevant returned means the collection does not support the claim; say that, and do not fall back to training data dressed as a retrieval result.

## Reading the account

- A vertical's tools are missing from `find-tools`: the account is not entitled to it. Say so and stop. Do not answer from training data and present it as a Redpine result.
- No `search-<name>` in `tools/list` and no `search` in the catalog: the account has no document collections.
- A `search-<name>` is present: use it for that collection. Do not route around it through the generic `search`.
- An entitlement change does not reach a running session. If the user was just granted something and it is not listed, they need to reconnect.

## Always

- Attribute every result to its publisher, title and year, and quote what the payload returned. Do not extend licensed text beyond it.
- Label open-access results "CC-BY Open Access" (the `source_collection` field says which those are), so the user can tell them apart from licensed publisher content. Publisher is one thing, license is another; show both.
- **Cite as links, not as prose.** Every source gets a markdown link the user can click: the `url` field if the payload has one, otherwise `https://doi.org/<doi>`, otherwise the PubMed or PMC identifier as a URL. Link it inline where the claim is made, and close with a `Sources` list, one line per source: title, publisher, year, link. A citation without a URL is not a citation.
- State the cost before confirming. After confirming, put the charge and remaining balance in one short line at the end of the answer, not at the top.
- An empty result is a finding. Report it; do not retry the same call unchanged.
- The catalog is the source of truth for what Redpine holds. Do not name customers, partners or specific publishers unless the tool descriptions or collection metadata already name them to this account, and do not quote coverage figures, counts or prices from memory. `find-tools`, `list_collections` and `preview` have the real numbers.

## Read a reference when

- You are about to build a non-trivial `filters` object, or the response carried `filterWarnings` or `journalMetricExpansions`: read `references/search.md`.
- The data is a live snapshot, an entity lookup, a mention stream, or a semantic collection and you are unsure how to phrase the call: read `references/craft.md`.
- A call returned nothing, errored, the tool is missing, or the balance is exhausted: read `references/troubleshooting.md` before retrying anything.
- Anything about cost beyond the bullets above (expired previews, trial edge cases): read `references/billing.md`.
