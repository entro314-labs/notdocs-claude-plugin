---
name: docs-health
description: Check a notdocs project for broken links, markdown that renders wrong and documents that fail validation, and repair what can be repaired. Use when the user asks to check, audit, clean up or fix their docs, asks about broken links or broken tables, has just moved or imported documents, or wants a document validated.
---

# Docs health

Three checks, all of which only read: links, markdown and validation. Run the checks first, show
what they found, then repair what the person agrees to.

`projects_list` first, and settle which project. Each check takes the `projectId` and, optionally,
a `branch`.

## 1. Links

`links_check` finds every link, image and wikilink on the branch that reaches nothing. It returns
how many documents it `checked` and the `broken` ones, each with its `docId`, `path`, `line`,
`kind` (`wiki`, `path` or `image`), the `target` as written, and `names`, what the target resolves
to.

- Anchors and links to other sites are not checked. Say so if the person asked for all links.
- When `truncated` is true the repository could not be listed whole and some links may be reported
  wrongly. Tell the person before fixing anything on that result.

To fix a broken link:

1. `docs_get` the document it is in, and keep the `hash`.
2. Work out which file the link meant. `docs_list` shows where documents are now.
3. Fix the link with `docs_update`, passing the hash you read as `baseHash`.
4. When you cannot tell what a link meant, leave it and say so.

Several broken links in one document are one update: read once, fix them all, write once.

## 2. Markdown

`markdown_check` finds markdown that renders as text instead of what it is. Each problem has its
`docId`, `path`, `line`, a `kind`, a `message` and `fixable`:

- `table`: pipe rows that are not read as a table, because the delimiter row has more or fewer
  cells than the header, or is missing.
- `tree`: a file tree written outside a code block.
- `mermaid`: Mermaid source written as text, or in a code fence with no language.
- `fence`: a code block that holds a line opening another. It comes with `fixable: false`, since
  only a person can tell which fence was meant, and nothing else in that document is reported
  until it is fixed.

An `.mdx` is not checked.

`markdown_fix` repairs every fixable problem on the branch in one commit. It takes no list of
documents: it is all of them or none. So:

1. Run `markdown_check` and show the person what it found, by document.
2. Ask before running `markdown_fix`.
3. It answers with the documents it changed and what it repaired in each. Only the lines at fault
   change.
4. If it fails with `conflict`, the documents kept changing while it ran. Run it again.
5. Run `markdown_check` once more. What is left should be the `fence` problems: show the person
   the document and line for each.

To repair a single document and leave the rest, edit it with `docs_update` instead.

## 3. Validation

`docs_validate` checks one document, by `id`: its frontmatter against its type's schema and, for a
business or legal document, its structure. It returns `ok` and the `issues`, each with a `path`, a
`message` and sometimes a `severity`.

- An issue with `severity` "warning" is advice and does not make `ok` false. One with no severity
  is an error.
- To validate a folder or a type, `docs_list` with a `path` prefix or a `type`, then validate each
  document. Say how many you checked.

To fix a frontmatter issue, `docs_get` the document and send the corrected `frontmatter` with
`docs_update`. It replaces the whole frontmatter, so send every key the document should keep.
Correct a key only when the right value is clear from the document; otherwise report it.

## What not to do on the way

- Do not move, rename or delete documents, or change a document's `type`, as part of a health
  check. Suggest it and let the person decide. When they do want a move, `docs_move` rewrites the
  links to the document in the same commit.
- `config_survey` finds the markdown in the repository that the workspace does not read. Report
  what it finds; the connector cannot change the repository's config.

## Finish

Report in short lists: links fixed; links left and why; markdown repaired, and the `fence`
problems left for a person; validation errors fixed and those still open.

## With a read-only connection

All three checks work. `docs_update` and `markdown_fix` are not offered: give the report and the
proposed fixes, and say that the connection reads only.
