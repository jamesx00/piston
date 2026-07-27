---
name: savepoint
description: Write or recall a savepoint — a dated summary of session work (done, learned, completed, not completed, next steps) stored in notes/savepoints/. Use when the user asks to "write a savepoint", "save progress", "checkpoint this", or asks about the "last savepoint" / "where did we leave off".
---

# Savepoint

A savepoint is a markdown snapshot of in-progress work on this repo, written to
`notes/savepoints/`, so a future session (or a different person) can pick up
where the work left off. This is project-local notes, not the memory system —
it captures ephemeral task state that memory deliberately excludes.

## Creating a savepoint

Triggered by: "write a savepoint", "save our progress", "checkpoint this",
or similar.

1. Reconstruct what happened in the session so far: what was attempted, what
   succeeded, what failed, what's still open. Pull from conversation context
   first; use `git status`, `git log --oneline -10`, and `git diff` only to
   confirm/ground details, not as the primary source.
2. Write to `notes/savepoints/<YYYY-MM-DD>-<short-topic>.md` (create the
   directory if missing) using this structure:

   ```markdown
   # Savepoint — <date> — <topic>

   ## Goal
   <what we were trying to do>

   ## Completed
   <bulleted list — be specific: file paths, commit hashes, commands run>

   ## Learned
   <non-obvious facts discovered — quirks, gotchas, workarounds found and why
   they were needed>

   ## Not completed
   <what's still open, and why it stopped there (error, timeout, blocked on
   a decision, etc.) — be honest here, this is the most useful section for a
   cold pickup>

   ## Next steps
   <concrete next actions, in order>

   ## Branch state
   <current branch, whether it's pushed, whether the working tree is clean>
   ```

3. Use one file per topic/session, not one growing file — old savepoints stay
   as history. If the user names a topic, use it as the slug; otherwise infer
   a short one from the session's main task.
4. Do **not** git add/commit/push the savepoint automatically — creating the
   file is a local, reversible action, but committing/pushing is not. Tell
   the user it's written and ask, or wait for them to say "commit and push"
   (as with any other change), same as the standing rule on risky actions.

## Recalling the last savepoint

Triggered by: "what's the last savepoint", "where did we leave off", "check
the savepoint", or similar — in this repo, always answer by reading the file,
never from memory of a prior conversation.

1. List `notes/savepoints/` and pick the most recent file. Filenames are
   date-prefixed (`YYYY-MM-DD-*.md`), so sort by filename, not by mtime
   (mtime reflects checkout time, not when it was written) — but if the
   directory doesn't exist or is empty, say so rather than guessing.
2. Read the full file and summarize it for the user (don't just dump it
   verbatim unless asked) — lead with **Not completed** and **Next steps**,
   since those are what matters for resuming work.
3. If the user asks for a specific past savepoint (by date or topic), list
   `notes/savepoints/` and match against that instead of assuming "last".
