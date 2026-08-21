---
name: smtc
description: >
  "Show Me The Code" — code-first reporting after coding work. Use this skill when
  summarizing any completed implementation, bug fix, refactor, redesign, or similar
  code change. Replaces prose-heavy explanations with short sentences, diff snippets,
  and file:line references so engineers can dissect the change directly.
license: MIT
metadata:
  author: twocommits
  version: "0.3.0"
---

# SMTC — Show Me The Code

After finishing coding work, do NOT write paragraphs explaining what you did.
Engineers read code faster than prose. Report with code.

## Response structure

1. **One-line outcome.** What changed, in a single short sentence.
2. **One-line why.** Root cause (fixes) or design choice (features/refactors). One sentence, no more.
3. **Changed files.** Bullet list, `path/to/file.rs:123` format (clickable), one fragment per file saying what changed there.
4. **Key snippets.** The load-bearing code only — the diff hunks a reviewer must see. `-`/`+` diff format for edits; plain code blocks for new files/functions. Skip boilerplate, imports, test scaffolding unless they ARE the change.
5. **Verification.** One line: what was run, result. E.g. `cargo test -p eddytor-engine → 214 passed`.
6. **Breaking changes.** Only when they exist: a `**BREAKING:**` line per break (API signature, config key, migration, wire format). Never folded into prose.

## Levels

Default: **compact**. Switch by saying `smtc annotated` / `smtc compact` / `smtc raw`.
Level holds for the rest of the session.

| Level | Behavior |
|-----------|----------|
| annotated | Code mandatory, prose allowed. A short context paragraph OK. For gnarly changes or explaining to others. |
| compact   | The response structure above. Default. |
| raw       | No prose sentences. Outcome fragment + files + diffs + verification. No why-line. |

Every level: `**BREAKING:**` lines, test failures, caveats, and open decisions stay in
words. Never compressed away — not even at `raw`.

## Flow diagrams — on request only

Do NOT include mermaid diagrams by default. Only when the user asks
("show flow", "diagram this", etc.). Then: mermaid `flowchart` or
`sequenceDiagram`, nodes = functions/components, edges labeled with what
moves (request, batch, token). Keep under ~10 nodes.

## Rules

- Sentences: short, declarative, max ~12 words. Fragments fine.
- No "I have successfully...", no "This ensures...", no restating the request.
- Snippets over descriptions. If you're about to describe code behavior in words, paste the code instead.
- Annotate snippets sparingly: a `// <-- the fix` marker beats a paragraph.
- Tradeoffs, caveats, and things the user must decide: still stated in words. Brevity never hides a problem.
- Test failures, skipped steps, unverified claims: report plainly, never buried.

## Example

> Fixed token expiry off-by-one in auth middleware.
> Strict `<` kept tokens valid at the exact expiry second.
>
> - `crates/common/src/auth/validate.rs:88` — comparison flipped
> - `crates/common/src/auth/tests.rs:210` — regression test added
>
> ```rust
> // validate.rs:88
> -if claims.exp < now {
> +if claims.exp <= now {
> ```
>
> `cargo test -p eddytor-common auth → 31 passed`

## Anti-example (do not do this)

> "I've successfully implemented the fix for the token expiration issue. The problem
> was that the comparison operator in the validation logic was using a strict
> less-than comparison, which meant that tokens were still considered valid at the
> exact moment of their expiration timestamp. I updated this to use less-than-or-equal
> so that..."
