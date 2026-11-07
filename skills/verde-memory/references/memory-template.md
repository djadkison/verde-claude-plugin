# Writing a Verde document

One document = one fact or decision. If an update covers two things, write two documents.

## Fields for `propose_memory`

| Field | What to put there |
|---|---|
| `title` | A declarative sentence that states the fact. "Deploys happen Tuesdays after standup", not "Deploy notes". Titles are how searches and `[[Title]]` cross-references find the document. |
| `content` | Markdown. Lead with the fact, then why it matters, then how to apply it. Keep it self-contained: a reader six months from now should need nothing else. |
| `type` | One of: fact, decision, policy, procedure, preference, project, person_role, event, working_note, rejected_approach, lesson_learned. |
| `tags` | Up to 20 short topic tags for filtering, e.g. `pricing`, `onboarding`, `deploy`. |
| `source_description` | Where this came from: "Decided in the 2026-09-04 pricing call", "Pasted from the support runbook", "Stated by the user in chat". Others use this to judge provenance. |
| `effective_date` / `expires_date` | ISO dates, only when the knowledge has a lifespan. Convert relative dates ("next quarter") to real dates first. |
| `bucket` | Name or id from `list_buckets`. |
| `visibility` | Omit to inherit the bucket's default. Set only to make the document narrower (e.g. a private note in a company bucket). |
| `supersedes_id` | Set when this document replaces an existing one — or use `supersede_memory` instead, which does the linking for you. |

## Body shape

```markdown
The Team plan costs $29 per seat per month, effective 2026-03-01.

**Why:** the $19 price did not cover support costs once the SLA was added.

**How to apply:** quote $29 in all new proposals; existing customers keep $19 until their renewal date. Supersedes [[Team plan pricing: $19/month per seat]].
```

## Cross-references

Write another document's exact title inside double square brackets — `[[Team plan pricing: $19/month per seat]]` — and Verde turns it into a navigable link. `get_memory` returns these as `references` (outgoing) and `referenced_by` (incoming), each with an id you can fetch directly.

## Reader pages (optional)

For a page written for people to read — a handbook chapter, an announcement, a runbook — pass `display: "canvas"`. Canvas documents get a reader layout and may use these markdown directives; ordinary documents should not:

- `:::callout{kind=note|success|warning|danger}` … `:::` — a highlighted aside
- `:::steps` around a numbered list — a step-by-step rail
- `:::decision{status=accepted date=YYYY-MM-DD}` with `### Context` / `### Decision` / `### Consequences`
- `:::stats` containing `::kpi{label="…" value="…" delta="+…"}` lines
- `:::details{summary="…"}` … `:::` — a collapsible section

Never emit raw HTML; it is stripped.
