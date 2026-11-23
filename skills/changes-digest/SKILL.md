---
name: changes-digest
description: Summarize what changed in a notdocs project since a point. Use when the user asks what changed, what is new, what happened since a date, a meeting or the last time they looked, who edited a document, or for the history of one document.
---

# A digest of what changed

Every change to a document is an event in the project's change feed. Reading the feed is how you
learn what changed without reading every document again. Both feed tools only read, so this works
with a read-only connection.

## What an event says

- `seq`: its number in the feed. Cursors are these numbers.
- `kind`: `created`, `changed`, `deleted` or `moved`. A `moved` event has `fromPath`, where the
  file was before.
- `path`, `docId` and `branch`.
- `hash`: the version of the document the event recorded, when there is one.
- `actorName`: who made it, when known: a person's email address, or `<token name>@agent` for an
  agent's token.
- `at`: when, as an ISO timestamp.
- `sections`: on some `changed` events, the headings whose sections differ, as slugs. An empty
  string stands for the text before the first heading.

## Pick the starting point

The feed is read by cursor. Neither tool takes a date.

**Since a date or a time** ("since Monday", "this week"):

1. `projects_list`, and settle which project.
2. `changes_tail` returns the newest events, newest first.
3. While the events are still after the point and `more` is true, call `changes_tail` again with
   `before` set to the `oldest` it returned. A page can be short or empty while `more` is true;
   keep going.
4. Keep the events whose `at` is after the point.

**Since last time**, when you have a cursor from an earlier digest:

1. `changes_since` with `since` set to that cursor returns the events after it, oldest first.
2. Continue from the `cursor` it returns while `more` is true. A page can be empty while `more`
   is true; keep going.

**The history of one document**: `changes_tail` with its `docId`.

Pass `branch` to read one branch. Left out, the feed tools return events from every branch of the
project, and each event names its own.

## Read what the changes were

The feed says which documents changed. To say what changed in them, read the documents.

1. Group the events by document. Several events on one document are one line in the digest.
2. For the documents that matter to the person, `docs_get` returns the current text.
3. To compare with an earlier version, `docs_blob` with a `hash` from an event returns the text of
   that version. The version before a change is the `hash` of that document's previous event.
4. Use `sections` to go straight to the parts that differ.

Do not fetch every changed document when there are many. Read the ones the person is likely to
care about and list the rest by path.

## Write the digest

- Start with the span it covers and the count: "14 changes to 6 documents since Monday".
- **Changed**: one line per document with its title, who changed it, and what changed in a few
  words. Name the sections when you have them.
- **New**: the documents created, with their type.
- **Moved or deleted**: old path and new path, or the path that went.
- Use the document's title and path. Say who by `actorName`; an event without one has no known
  actor.
- Report what the feed and the documents show. Do not guess at why a change was made.

Both feed tools return a `cursor`, the point `changes_since` continues from. End the digest with
the one from your newest read, so the next digest can start there: "Next time, ask for changes
since cursor 482."

## If the feed is long

`limit` sets the page size, up to 500 (100 when left out). When the span holds more events than
is useful to read, say how far back you went and offer to narrow it to a folder (filter the
events by `path`), a branch, or one document.
