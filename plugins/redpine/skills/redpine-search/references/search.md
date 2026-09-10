# Using `search` and `search-<name>` well

Both take the same arguments. `search` needs a `collection` from
`list_collections`; `search-<name>` has it bound. The authoritative field list
for `filters` lives in the tool's own schema. Read it with `inspect-tool`; it
is long and it differs by collection. What follows is how to use it, not what
is in it.

## Query

- A sentence stating the actual question, up to 1000 characters. Semantic
  retrieval ranks by meaning; a list of keywords retrieves worse than prose.
- Constraints go in `filters`, not in the query text. "Nature papers after
  2020 on X" is `query: "X"` plus a journal and date filter. Leaving the
  constraint in the prose makes it a soft preference the ranker may ignore.
- `limit` is 1 to 30, default 10. Preview at the default, then unlock
  selectively, rather than asking for 30 up front.
- `include_figures` adds latency. Set it only when the question is about a
  figure or visual data.

## Filters: which field to reach for

- **A named journal.** Filter on its identifier field, not its name. One
  journal appears under several spellings; the identifier survives them all.
  The schema names the field and the formats it accepts.
- **A subject across publications.** Look for a cross-brand subject field
  before filtering on journal. If the collection has one, it is the right
  axis for "by topic" questions.
- **A kind of article.** Filter on article type (reviews, case reports,
  protocols) or on section (methods, results) when the user wants a kind of
  evidence rather than a topic. Section filtering is how you answer "what
  methods did they use" without reading every chunk.
- **A date window.** ISO `YYYY-MM-DD`; ranges are detected in any range
  operator.
- **Excluding.** There is no separate exclusion syntax. Negate with `not`
  in the flat form, or `ne` / `not_in` in the structured form.
- **Journal-level metrics.** Range operators only. The metric is OpenAlex
  two-year mean citedness. It is **not** the Journal Impact Factor and must
  not be described as one. A journal with no value is absent from results,
  not scored zero.

## Two filter formats, never mixed

Flat key-value (preferred; top-level keys are ANDed):

```json
{"publisher": ["Elsevier", "Springer"],
 "publication_date": {"gte": "2020-01-01", "lt": "2024-01-01"}}
```

Structured DSL, only when you need OR or nesting:

```json
{"or": [{"field": "journal", "eq": "Nature"},
        {"field": "journal", "eq": "Science"}]}
```

The server picks the format from the top-level keys. Combinator or `field` at
the top means DSL; anything else means flat. One request, one format.

## Read the response, not just the results

- `filterWarnings`: you filtered on a field that is not indexed for this
  collection. The call still ran, as a full scan. Rephrase against an indexed
  field, and tell the user the filter was not exact.
- `journalMetricExpansions`: what a metric threshold resolved to. Quote it
  when the user asked for "high-impact" anything, so they see what that meant
  in practice.
- `query_id`: keep it. The same results re-fetch free for seven days.
- Each chunk carries its source metadata. Cite from it, as a link. Build the
  URL from whichever identifier the payload provides, in this order:
  `url` as given; `https://doi.org/<doi>`;
  `https://pmc.ncbi.nlm.nih.gov/articles/<pmcid>/`;
  `https://pubmed.ncbi.nlm.nih.gov/<pmid>/`. Do not invent a URL when none of
  those is present; cite title, publisher, and year and say the record has no
  resolvable link.
- Shape of a finished answer: claims with inline links where they are made, a
  `Sources` list at the end (title, publisher, year, link, one per line), and
  the charge and remaining balance as the final line.
