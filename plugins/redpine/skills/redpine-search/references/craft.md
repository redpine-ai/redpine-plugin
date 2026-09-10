# Querying well

Organised by what the data looks like, not which provider serves it. A new
integration or collection fits one of these shapes; the advice does not change.

## Live snapshots

Positions, prices, statuses: anything that is true at a moment.

- Timestamp every value you report. The payload has the moment; use it.
- Do not describe a snapshot as the present tense. "Was at X at 14:02Z", not
  "is at X".
- Units and coordinate order vary across providers. Take them from the schema
  and the payload. Never convert silently.

## Entity lookups

Companies, people, places, identifiers.

- Resolve names to codes or identifiers first. A city is several airports; a
  company name is several tickers across jurisdictions; a person's name is many
  people. `inspect-tool` tells you which identifier the tool wants.
- Two identifier systems that look alike are usually not interchangeable. Do
  not translate between them on assumption.
- When the question is ambiguous, say which entity you searched for.

## Monitoring and mentions

Coverage, press, brand tracking: anything with a stream over time.

- Anchor to an entity and a window. A bare name returns the firehose.
- Counting appearances and reading substantive coverage are different
  questions. Pick the tool that answers the one asked, not the one that returns
  more.
- Licensed text is quoted, attributed, and not extended beyond what was
  returned.

## Semantic collections

Full-text document collections searched by meaning.

- Query in a sentence that states the actual question. A keyword-stuffed query
  retrieves worse than plain prose against a semantic index.
- Use `filters` for publisher, journal, and date rather than stuffing them
  into the query text.
- A result is a chunk of a document, never the document's conclusion.
- Cite from the payload (title, publisher, date) and never reconstruct a
  citation from memory.
- Publisher metadata dates can be wrong. If the age of a source carries the
  argument, report what the record says rather than asserting it.
- Nothing relevant returned means the collection does not support the claim.
  Say that. Do not fall back to training data and present it as a retrieval
  result.
