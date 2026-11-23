# notdocs for Claude

[notdocs](https://notdocs.dev) is a workspace whose documents are markdown files in your git
repository. This plugin connects Claude to your notdocs account and teaches it how the workspace
is meant to be used: read a document before changing it, let an update merge with edits made in
the meantime, and work through specs, tasks, comment threads and the change feed with the tools
notdocs has for them.

It works in Claude on the web, desktop and mobile, in Cowork, and in Claude Code.

## What's inside

- **The notdocs connector** (`.mcp.json`): the remote MCP server at `https://notdocs.dev/api/mcp`.
  It lists, reads and searches the documents of every project you can open, runs query blocks,
  reads comment threads and the change feed, checks links and markdown, and validates and exports
  documents. With write access it also creates, updates, moves and deletes documents, and adds,
  answers and resolves comments.
- **Skills**
  - `notdocs` - how to work with a notdocs workspace: projects and branches, document types, read
    before write, how an update merges and what a conflict means.
  - `plan-from-spec` - read a spec and create a task document for each piece of work it states.
  - `review-comments` - work through the open comment threads on a document: make the change,
    reply, resolve, and leave open what needs a person.
  - `changes-digest` - what changed in a project since a date or since last time, from the change
    feed.
  - `docs-health` - find links that reach nothing, markdown that renders wrong and documents that
    fail validation, and repair what can be repaired.

## Use it

Install the plugin, then connect the notdocs connector from the plugin's **Connectors** tab. Claude
opens notdocs, where you sign in and choose what Claude may do: read, or read and write. In Claude
Code the connector comes with the plugin; run `/mcp` and pick notdocs to sign in.

Then ask for what you need:

- "Read the spec at `docs/specs/billing.md` and draft the tasks it implies."
- "Work through the open comments on the onboarding guide."
- "What changed in the handbook since Monday?"
- "Check our docs for broken links and tables that don't render, and fix what you can."
- "Which tasks are blocked?"

You need a notdocs account. Claude connects to your account, so it reaches every workspace you can
open.

## What Claude can and cannot do

- You choose read, or read and write, when you connect. A connection granted read is offered only
  the tools that read. To allow writing later, connect again and choose read and write.
- Each tool says whether it only reads and whether it can overwrite or remove something, so Claude
  can run the reads and ask you before a change.
- An update names the version Claude read. If the document changed since, notdocs merges the two
  edits; when they collide, the conflict comes back to Claude and nothing is written.
- An `.mdx` file is read but never written.
- Claude cannot manage organizations, people, billing or tokens. Those tools are not offered to a
  connected app, whatever you could do yourself.

## Data

The plugin holds only instructions and the connector's address. It runs no code on your machine
and stores nothing.

Tool calls go from Claude to notdocs at `notdocs.dev`, signed in as you, with the access you
allowed. Your conversation stays with Claude: notdocs receives only the operations Claude calls.
What Claude reads becomes part of your conversation with Claude, which Anthropic handles under its
own terms.

A change Claude makes is recorded as yours. In a repository on GitHub it is a commit in your name,
with the app named in a `Via:` line of the commit message. notdocs keeps the app's name and the
addresses it registered, the access you allowed, when you connected it and when it was last used,
and hashes of its tokens.

The plugin adds no service of its own. How notdocs handles your documents, its index and the
providers it uses is in its [privacy policy](https://notdocs.dev/legal/privacy), under "Connected
apps".

To disconnect, open Settings › Connected apps in notdocs, which ends the app's access at once, or
remove the connector in Claude.

## Install in Claude Code

```
/plugin install notdocs --marketplace entro314-labs/notdocs-claude-plugin
```

## Support

Documentation: [notdocs.dev/agents](https://notdocs.dev/agents). Write to support@notdocs.dev.

## License

MIT. See [LICENSE](LICENSE).
