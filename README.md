# SMTC — Show Me The Code

An agent skill that makes coding assistants report finished work with **code, not prose**.

After a feature, bug fix, or refactor, the agent answers with:

- one-line outcome
- changed files as `path/to/file.rs:123` bullets
- diff hunks of the load-bearing code
- one-line verification (`cargo test → 214 passed`)

instead of paragraphs like *"I've successfully implemented the fix for…"*.

Three levels — `annotated` (code + guiding prose), `compact` (default, structured),
`raw` (diffs only, no sentences). Switch mid-session: "smtc raw".

The whole skill is a single file: [`SKILL.md`](SKILL.md). The YAML frontmatter is
Claude Code metadata; every other tool just consumes the markdown body.

## Install

### Claude Code

Personal (all your projects):

```sh
mkdir -p ~/.claude/skills/smtc
curl -fsSL https://raw.githubusercontent.com/2commits/smtc/main/SKILL.md \
  -o ~/.claude/skills/smtc/SKILL.md
```

Per project (committed, whole team gets it):

```sh
mkdir -p .claude/skills/smtc
curl -fsSL https://raw.githubusercontent.com/2commits/smtc/main/SKILL.md \
  -o .claude/skills/smtc/SKILL.md
```

Claude Code applies it automatically when summarizing coding work; `/smtc` invokes it explicitly.

**Always-on (recommended):** skill invocation is model-discretionary and long sessions can
compact the loaded rules away. A `SessionStart` hook injects them unconditionally:

```sh
mkdir -p ~/.claude/hooks
cat > ~/.claude/hooks/smtc-sessionstart.sh <<'EOF'
#!/usr/bin/env bash
SKILL="$HOME/.claude/skills/smtc/SKILL.md"
[ -f "$SKILL" ] || exit 0
echo "SMTC MODE ACTIVE — code-first reporting. Apply these rules to every coding-work summary this session:"
sed '1,/^---$/d' "$SKILL"
EOF
chmod +x ~/.claude/hooks/smtc-sessionstart.sh
```

Then add to `~/.claude/settings.json`:

```json
"hooks": {
  "SessionStart": [
    { "hooks": [ { "type": "command", "command": "bash \"$HOME/.claude/hooks/smtc-sessionstart.sh\"" } ] }
  ]
}
```

The hook reads the installed skill file, so re-running the install `curl` updates both paths.

### Cursor

```sh
mkdir -p .cursor/rules
curl -fsSL https://raw.githubusercontent.com/2commits/smtc/main/SKILL.md \
  -o .cursor/rules/smtc.mdc
```

Then edit the frontmatter of `.cursor/rules/smtc.mdc` to Cursor's format:

```yaml
---
description: Code-first reporting after implementing, fixing, or refactoring
alwaysApply: true
---
```

### Codex / Jules / anything reading AGENTS.md

Append the skill body to your `AGENTS.md`:

```sh
echo "" >> AGENTS.md
curl -fsSL https://raw.githubusercontent.com/2commits/smtc/main/SKILL.md \
  | sed '1{/^---$/!q;};1,/^---$/d' >> AGENTS.md
```

(the `sed` strips the YAML frontmatter)

### GitHub Copilot

Same as above, into `.github/copilot-instructions.md`.

### Any other LLM

Paste the body of [`SKILL.md`](SKILL.md) (everything below the frontmatter) into
your system prompt / custom instructions.

## Update

Re-run the same install command — it overwrites the file with the latest version.
For the append-based installs (AGENTS.md, Copilot), remove the old SMTC section first.

## License

[MIT](LICENSE) — maintained by [twocommits](https://github.com/2commits).
