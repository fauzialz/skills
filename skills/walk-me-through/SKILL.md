---
name: walk-me-through
description: Read a pasted source (article URL, raw text, or local file) and walk the user through it concept-by-concept in plain language, one chunk at a time, pausing after each so the user can ask questions or explore related tangents before continuing. Use ONLY when the user explicitly asks to be walked through or explained something — e.g. "explain this to me", "walk me through this", "eli5 this", "break this down for me". Do NOT trigger on a bare pasted link or article text with no such phrase — that's the user's call to make, not an assumption.
---

# Skill: walk-me-through

Turn a source into a paced, conversational explanation in plain language — never a
wall-of-text summary dumped in one message.

## 1. Resolve the source(s)

- URL → fetch it.
- Pasted raw text → use it directly.
- Local file (PDF/markdown/txt) → read it.
- Multiple sources given in the same message → treat them as **one combined pool** of
  concepts, not separate walkthroughs. Interleave and cross-reference where they overlap.
- A new, unrelated source pasted **mid-walkthrough** (not a follow-up question, a whole
  new thing) → stop and ask: finish the current walkthrough first, or pause it and switch?
  Never switch silently, never ignore it silently.

## 2. Segment into concepts

Break the source into a sequence of concepts, using the source's own structure as a guide,
not a strict boundary — merge or split sections where the natural conceptual unit doesn't
match the source's headings. Don't fix the total count up front; it can shift once tangents
get pursued.

## 3. Explain one concept at a time

For each concept:

- Explain it in plain, layman's language: smart-generalist depth, no unexplained jargon,
  analogies where they help. This depth is fixed — don't ask the user about their
  background first. If the user wants more depth on a specific point, they'll ask a
  follow-up; go deeper on just that point, not the whole chunk.
- **Format strictly**: lists with short descriptions, or short paragraphs (2-4 sentences).
  Never a long unbroken block of prose, no matter how many list items or paragraphs that
  produces.
- **Visuals**: when a flow, hierarchy, or comparison would genuinely help, show one.
  - If the `Artifact` tool is available in this session, publish a small rendered diagram
    (e.g. a mermaid flow) and share the link.
  - If it isn't available (bare CLI without that tool), draw a simple ASCII diagram inline
    instead.
  - Don't force a visual where a sentence or list would do.
- After explaining, if 1-2 genuinely relevant but under-explored related topics come to
  mind — drawn from your own knowledge, not a web search — offer them briefly. Skip the
  offer entirely if nothing good fits; never force a tangent onto a chunk that doesn't
  have one.
- Then **stop and wait**. Don't ask "ready for the next one?" — just stop. Any reply is
  handled naturally:
  - A plain continue signal ("ok", "go on", "next") → move to the next concept.
  - A question or tangent pick → answer it, going as deep as needed.
  - Resuming the main thread after that → a short bridge sentence back to the outstanding
    topic ("back to concept 3: ...").

## 4. Tangents

- Offering a tangent costs nothing (no search) — it's just a name/description from your
  own knowledge.
- **Only when the user actually chooses to pursue one**, do a web search to ground the
  explanation in current, accurate information before explaining it. Never search for a
  tangent that wasn't picked.
- A pursued tangent becomes its own branch: same depth rules, same pacing rules, and it
  can itself offer further tangents (recursive). Track the path.
- A tangent the user doesn't pick is **parked**, not dropped — remember it and re-offer it
  once everything else (main walkthrough + any pursued branches) is finished.

## 5. Progress footer

Every message ends with a one-line breadcrumb showing exactly where the conversation is:

```
📍 Main 3/7 → Branch: Memory architectures → Branch: Vector DBs (1/1) | parked: Tool-calling protocols
```

- Always show the main-topic progress.
- Append each active branch level as `→ Branch: <name> (<progress>)`.
- Append `| parked: <topics>` whenever there are declined-but-remembered tangents.
- Omit a segment when it's empty (no branches yet, nothing parked yet).

## 6. Wrap-up

Once the main walkthrough and all pursued branches are finished:

1. Re-offer any parked tangents, one round — the user can still pursue or drop them.
2. Proactively (but optionally — never automatic) offer to save a short synthesis note of
   what was covered:
   - If the current project looks second-brain-shaped (e.g. an `ingest` skill or a
     `Knowledge/` wiki structure is present), offer to route the note through `ingest`.
   - Otherwise, offer a plain summarized `.md` file, and ask where to save it — there's no
     fixed default location, always ask.
   - If declined, just end — no note is written.
