---
name: plan-from-spec
description: Turn a spec in notdocs into task documents. Use when the user asks to plan from a spec, PRD or RFC, to break a spec into tasks, to draft the tasks a document implies, or to put the work for a spec on the board.
---

# Planning tasks from a spec

A task in notdocs is a document of type `task`: a markdown file with a `status` in its
frontmatter. This skill reads a spec and creates one task document for each piece of work the spec
states.

## Read the spec

1. `projects_list`, and settle which project the spec is in.
2. `docs_get` the spec. Pass what the person gave as `path` when it looks like a path
   (`docs/specs/billing.md`), otherwise as `id`. If they named it loosely, find it with
   `docs_list` and `type` "spec", or with `search`.
3. Read the whole spec before drafting anything.

## See what already exists

1. `docs_list` with `type` "task" shows the tasks on the branch. Do not create a task that is
   already there.
2. `docs_backlinks` with the spec's `id` shows the documents that already link to it, which is
   where earlier tasks for this spec will be.
3. `config_get` returns the docs root (`docs` unless the repository set another). Create the tasks
   under it; a path outside the workspace is refused with `invalid`.

## Draft the tasks

- One task for each piece of work the spec states. Do not invent scope the spec does not state.
- Keep each task small enough to finish in one sitting. Split what is larger.
- A short imperative title, such as "Add the invoice export".
- If the spec leaves a decision open, do not turn a guess into a task. Say what is undecided.

When the spec implies more than a handful of tasks, list the titles for the person first and
create them once they agree.

## Create them

For each task call `docs_create` with:

1. `template`: "task". It sets the type and a `status` of `todo`.
2. `path`: `docs/tasks/<slug>.md` unless the person named another folder. New documents end in
   `.md`.
3. `title`: the short imperative title.
4. `body`: the text of the document. A `body` you send replaces the template's, so write all of
   it:

   ```markdown
   # Add the invoice export

   From [Billing](../specs/billing.md).

   ## Goal

   What this task achieves, in a sentence or two.

   ## Done when

   - [ ] A criterion someone can check
   ```

5. `frontmatter`, only for what the spec or the person gave you. The task schema knows `status`
   (`todo`, `doing`, `blocked`, `review`, `done`), `priority` (`p0` to `p3`), `assignee`, `area`,
   `due` and `blockedBy`. Leave out what nobody stated.

Link each task back to the spec with a relative markdown link, as in the body above.

## When a create fails

- `conflict`: a document already exists at that path. Read it; if it is the same task, leave it,
  otherwise choose another slug.
- `invalid`: the path is outside the workspace, or the frontmatter does not match the task
  schema. The `issues` name the keys at fault.
- `not_found`: the repository has no template of that name. `templates_list` shows the ones it
  has.

## Finish

Report the list of paths you created, the tasks you skipped because they already existed, and
anything in the spec that was too open to plan.

## With a read-only connection

`docs_create` is not offered. Read the spec and the existing tasks as above, give the drafted
tasks in the conversation, and say that the connection reads only.
