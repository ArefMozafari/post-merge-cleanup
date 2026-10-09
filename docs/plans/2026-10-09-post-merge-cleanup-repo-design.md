# post-merge-cleanup as a cross-agent skill repository

Date: 2026-10-09

## Goal

Publish a personal skill for cleaning up after a merge so it installs in the widely used coding
agents, the same way [pr-evidence](https://github.com/ArefMozafari/pr-evidence) does.

## Decision

One repository with the pr-evidence layout, so each agent's installer finds the same `SKILL.md`:

```
.claude-plugin/marketplace.json     Claude Code marketplace; Copilot CLI reads it too
.claude-plugin/plugin.json          Plugin manifest; the plugin root is the repository root
.cursor-plugin/marketplace.json     Cursor marketplace
.cursor-plugin/plugin.json          Cursor plugin manifest
gemini-extension.json               Gemini CLI extension manifest
skills/post-merge-cleanup/SKILL.md  The skill; `skills/*/SKILL.md` is what npx skills and gh skill discover
```

The reasons behind that layout, and the Cursor manifest constraints, are recorded in the
pr-evidence design document and hold here unchanged.

### Name

The personal skill was called `closing-merged-units`. "Unit" was private vocabulary for one
self-contained piece of work; the public name `post-merge-cleanup` says when it runs and what it
does.

### Generalized from the personal skill

- The `user-invocable` frontmatter key is gone; the Agent Skills reference validator rejects it.
- "Unit" became "change" throughout.
- The memory section assumed one agent's memory layout: `[[wiki-links]]`, an index file with a
  hook per line, and `rg` and `Read` by name. It is now "notes and memory", skipped when the
  agent keeps none, and speaks of links, index entries, and searching the directory.
- Called with no branch or number, the skill now lists the local branches `git branch -vv` marks
  `gone` and takes them one at a time.

The merge confirmation, the git steps and their stop points, the four fates of a claim, and the
four-list report are unchanged.

## Alternatives rejected

- **One collection repository for several skills.** Each skill should install on its own, and the
  pr-evidence layout is already proven in each agent; repositories link to each other instead.
- **Dropping the notes and memory section.** Stale notes are half the residue a merge leaves;
  making the section conditional keeps it for agents that keep notes.

## Verification

- `agentskills validate`, `claude plugin validate --strict` on both Claude Code manifests, the
  Gemini manifest field check, and the Cursor schema check at `cursor/plugins` commit `ecc249f`
  all pass locally.
- Installs checked against the local checkout: Claude Code (`claude plugin marketplace add` and
  `install` in a throwaway configuration folder; the skill lands in the plugin cache),
  `npx skills add` for the Codex and Cursor targets, and `gh skill install --from-local`.
- Gemini CLI, Copilot CLI, and the Cursor CLI install from GitHub and are not installed on the
  machine used, so their README commands are checked after the first push.
