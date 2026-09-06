# skills

Personal Claude Code skills, packaged as a single installable plugin.

## Install

Every skill here is a plain folder + `SKILL.md` — Anthropic's open
[Agent Skills](https://agentskills.io) format. A growing list of agents read that format
directly, with no translation needed; everything else can still use the skill's
instructions, just via that agent's own mechanism instead.

### Claude Code

Plugin (recommended — tracks updates to this repo):

```
/plugin marketplace add fauzialz/skills
/plugin install fauzialz-skills
```

No-plugin route: copy a skill folder straight in.

```
cp -r skills/walk-me-through ~/.claude/skills/walk-me-through   # personal, all projects
cp -r skills/walk-me-through .claude/skills/walk-me-through     # this project only
```

Docs: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

### Gemini CLI

Same Agent Skills format — drop the folder in as-is, no reformatting:

```
cp -r skills/walk-me-through ~/.gemini/skills/walk-me-through   # or ~/.agents/skills/
cp -r skills/walk-me-through .gemini/skills/walk-me-through     # or .agents/skills/
```

Docs: [geminicli.com/docs/cli/skills](https://geminicli.com/docs/cli/skills/)

### OpenAI Codex CLI

Reads Agent Skills folders directly, same as above. It also reads a root `AGENTS.md` for
general instructions — see the fallback section below if you'd rather go that route.

### Cursor

Cursor doesn't read `SKILL.md` folders directly — it reads `.cursor/rules/*.mdc` (Markdown
+ YAML frontmatter: `description`, `globs`, `alwaysApply`; plain `.md` without that
frontmatter is ignored). Copy the skill's body into a new `.mdc` file:

```
mkdir -p .cursor/rules
cp skills/walk-me-through/SKILL.md .cursor/rules/walk-me-through.mdc
# then add/adjust the frontmatter to Cursor's format
```

Docs: [cursor.com/docs/rules](https://cursor.com/docs/rules)

### GitHub Copilot

Per-skill files (recommended — keeps skills separate, frontmatter can target specific
file globs):

```
mkdir -p .github/instructions
cp skills/walk-me-through/SKILL.md .github/instructions/walk-me-through.instructions.md
```

Or merge everything into the single repo-wide `.github/copilot-instructions.md` if you'd
rather have one file. Reusable prompts go in `.github/prompts/*.prompt.md`.

Docs: [docs.github.com — add repository instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions-in-your-ide/add-repository-instructions-in-your-ide)

### Windsurf

```
mkdir -p .windsurf/rules
cp skills/walk-me-through/SKILL.md .windsurf/rules/walk-me-through.md
```

(Legacy single-file `.windsurfrules` is still read if you prefer that instead.)

Docs: [windsurf.com/editor/directory](https://windsurf.com/editor/directory)

### Cline

```
mkdir -p .clinerules
cp skills/walk-me-through/SKILL.md .clinerules/walk-me-through.md
```

Docs: [cline.bot — .clinerules](https://cline.bot/blog/clinerules-version-controlled-shareable-and-ai-editable-instructions)

### Continue.dev

Reference the file as a rule in `config.yaml`, or copy it under `.continue/rules/`:

```yaml
rules:
  - skills/walk-me-through/SKILL.md
```

Docs: [docs.continue.dev/customize/deep-dives/rules](https://docs.continue.dev/customize/deep-dives/rules)

### Aider

Aider takes one merged conventions file rather than separate per-skill files:

```
cat skills/walk-me-through/SKILL.md >> CONVENTIONS.md
aider --read CONVENTIONS.md
```

Docs: [aider.chat/docs/usage/conventions.html](https://aider.chat/docs/usage/conventions.html)

### AGENTS.md — universal fallback

For any agent without a dedicated skills/rules mechanism, a root `AGENTS.md` is the
widest-supported fallback (read by Codex, Cursor, Copilot, Windsurf, Aider, and others).
It's one merged file, not a per-skill split, so append what you need:

```
cat skills/walk-me-through/SKILL.md >> AGENTS.md
```

Docs: [agents.md](https://agents.md)

## Skills

- **walk-me-through** — paced, conversational walkthrough of a pasted source (URL, text,
  or file), explained concept-by-concept in plain language, with optional related-topic
  tangents along the way.
