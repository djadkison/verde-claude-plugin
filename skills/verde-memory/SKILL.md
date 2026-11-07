---
name: verde-memory
description: Use when the user wants to save, find, update, or publish their team's shared knowledge in Verde — decisions, conventions, project facts, processes, lessons — or asks a question about their own team's projects, decisions, or history that Verde's memory should answer. Not for general-knowledge questions, calendar or scheduling tasks, or storing passwords, keys, or other secrets.
---

# Verde — recall, save, and keep team knowledge current

Verde is the team's persistent, shared memory, reached through the Verde plugin's tools. A **vault** belongs to a team and holds **buckets** (folders); buckets hold **documents**. Every document carries a visibility: **public** (on the open internet once published), **company** (the whole team), or **private** (the signed-in user only). A document may be narrower than its bucket, never wider.

Tools: `list_vaults`, `list_buckets`, `search_memories`, `get_memory`, `list_recent_memories`, `list_comments` (read); `create_bucket`, `update_bucket`, `add_bucket_fields`, `propose_memory`, `update_memory`, `set_properties`, `supersede_memory`, `archive_memory`, `publish_memory`, `add_comment` (write). Every tool takes `vault` as a `team/vault` slug from `list_vaults`.

## Facts you must not infer

- **Which vault or bucket** — always take it from `list_vaults` / `list_buckets`.
- **That something is not in Verde** — only `search_memories` can establish that.
- **Dates** — convert "next Friday" to the real date before saving.
- **That the user wants a public document published** — only an explicit approval in this conversation, after seeing the exact text, counts.

## Workflow 1 — Recall (a question about the team's projects, decisions, conventions, or history)

1. `search_memories` with a natural-language `query`: a short plain-words description of what you need — the topic and the kind of fact ("how we approve deploys", "Team plan pricing decision") — not a list of keywords or identifiers. Add `include_historical: true` when the user asks how something changed or what it used to be.
2. Results carry short excerpts, `status`, `is_current`, and a `url`. Pick the relevant hits and fetch full text with `get_memory` — pass several ids in `ids` in one call. Follow `references` / `referenced_by` ids the same way when they matter.
3. Answer from the documents. Name the document title, say when it was last updated, and distinguish the current version from superseded history explicitly ("current: $29/seat; superseded: $19/seat").
4. Share the `url` when the person would want to open the document itself (a design doc, decision record, plan). Skip it for a one-line fact.
5. A miss is not an answer. Before saying Verde has nothing, search two or three more times with different wording — synonyms for the key terms, the broader topic, a likely document title — and drop any filters you added. Only then say so plainly. Once the user settles the answer in conversation, offer to save it (Workflow 2).

Where a document contradicts your own assumption, the document wins. Where the document is clearly stale, say so and offer to supersede it (Workflow 3).

## Workflow 2 — Save (the user states something durable, or asks to remember it)

Durable = a decision and its reasoning, a stable preference or convention, a project fact, constraint, or deadline, a correction of a previous belief, a process or fix that worked.

1. **Get agreement.** Offer in one line ("Save this to Verde?") and wait for a yes — unless the user already asked Verde to save or remember it, or the connection has Automatic Memory on (see below).
2. **Check for duplicates.** `search_memories` for the topic. If a document already covers it, switch to Workflow 3 instead of creating a second one.
3. **Place it by description.** `list_vaults` (each vault carries a `description` of what belongs in it), then `list_buckets` for the vault (each bucket likewise). Choose the vault and bucket whose descriptions fit; do not ask the user where to file unless two fit equally, and then name the two. Call `create_bucket` only when nothing fits; it returns an existing bucket when a similar name already exists.
4. **Draft one document per fact** using [references/memory-template.md](references/memory-template.md): a declarative title that states the fact, a body with the fact, why it matters, and how to apply it, absolute dates, a `type`, `tags`, and `source_description`. Split compound updates into separate documents. Write `[[Exact Title]]` in the body to cross-reference another document in the same vault (links do not resolve across vaults).
5. **Call `propose_memory`.** Leave `visibility` at the bucket's default unless the user asked for narrower. If the bucket is public, the result is a **draft** — tell the user it is not live and that publishing is a separate confirmed step (Workflow 4).
6. **Report** in one or two lines: title, bucket, visibility, and status (draft or published). No link needed for routine saves.

If a prompt supplies the exact title and body, use them verbatim; do not ask clarifying questions.

### Automatic Memory (standing consent, on for new connections unless unticked)

**Automatic Memory** is a per-connection setting. New connections start with it on — the box is pre-ticked and visible on the consent screen — and the person can untick it there or switch it off later on the Verde dashboard (MCP Connections). It cannot be changed from chat. When it is on, `list_vaults` and `list_buckets` report `automatic_memory: true`, and you may skip step 1 for clearly durable, non-sensitive information: call `propose_memory` with `automatic: true`. The server keeps that narrow:

- an automatic save always becomes the user's own **private draft** — `visibility` and `publish` are ignored, nothing is shared or published
- **tell the user** in one line what you saved and that it is in their Drafts to keep, publish, or archive
- on a connection where Automatic Memory is off, an `automatic` save is **refused** and nothing is written — offer instead, and call again without the flag once they agree
- the "Never" list below applies unchanged

If a user asks you to "just save things without asking" on a connection where it is off, explain that this is a setting they turn on in Verde for the connection (MCP Connections), that it only allows private drafts, and offer to save the current item now.

## Workflow 3 — Keep it current

- Wrong or incomplete wording, refreshed tags or metadata → `update_memory`. Pass the `expected_version` you received from `get_memory` so you never overwrite someone else's newer edit. Readers see only the new text; the prior text is retained as a revision.
- The knowledge itself **changed** (a new price, a reversed decision, a replaced process) → `supersede_memory` with `old_memory_id`. Both documents stay in history and link to each other. Prefer this whenever the previous state matters.
- Obsolete with no replacement → `archive_memory`. It is retained and recoverable, just out of normal retrieval.
- `get_memory` shows `open_comments` when people left unresolved comments → read them with `list_comments` before editing; they are usually the change being asked for. After an edit that addresses a thread, `add_comment` with `reply_to` the thread id and `resolve: true`, saying what you changed. Leave threads you didn't fully address open.

- A document's **properties** (typed values under the fields its bucket declares — `fields` on `list_buckets`: a select like priority or delivery_state, a person like owner, a date, a list, a number, a checkbox, text) are the cheap way to record and change state: `set_properties` merges only the keys you pass (`null` removes one) and leaves the body alone; `propose_memory` and `supersede_memory` take `properties` too. Ask what properties a document needs; when its bucket does not declare one, add it with `add_bucket_fields` (add-only — existing fields, values, and documents are untouched until set), whether the bucket declares none yet or already has a few; when the document does not belong with the others, file it in a bucket that fits. Removing or retyping a field drops stored values and is done on the bucket's Properties page in Verde, not from chat. An undeclared key is refused and the error names the declared set. A `person` is set by name, email, or id and reads back as the name; someone not on the team is refused — suggest inviting them. Filter with `where` on `search_memories` / `list_recent_memories` ({"state": "todo"}, {"delivery_state": "live"}) to list by state without reading bodies. `required` fields block publishing, not drafts.

Before touching a document that is **published public**, tell the user that editing it changes its public web page immediately and archiving it removes the page immediately, and get their go-ahead.

## Workflow 4 — Publish a public document (only when the user asks)

1. `get_memory` the draft and show the user its exact title and full body.
2. Ask for explicit approval to put that text on the open internet.
3. Only after a clear yes in this conversation: `publish_memory` with `confirm_public: true`. Without the flag the tool refuses and nothing is published — that is the intended safeguard, not an error to work around.
4. Company and private drafts publish with a plain `publish_memory` call; they are visible only to signed-in members.

Superseding a public document always produces a draft; take it live with the same confirmed step.

## Never

- Save passwords, API keys, tokens, or other credentials — decline and explain that secrets do not belong in shared knowledge.
- Save transient session details, speculation stated as fact, or anything the user asked to keep off the record.
- Put personal or private information in a company or public bucket.
- Publish a public document, or edit or archive a live public one, on your own initiative.
- Duplicate a document that already exists — search first, then update or supersede.

## When to stop and ask

Ask before acting when the bucket is ambiguous between two reasonable choices, when a save would change visibility, or when a document you found looks stale but the user has not confirmed what replaced it. Otherwise proceed and report what you did.
