+++
title = "Multi-repo Claude Code: let each repository brief its own agent"
description = "Claude Code reads a repository's CLAUDE.md, rules, skills and hooks only when the session starts there. Instead of stretching one session across repositories, hand each change to a background session that lives in the repository it changes."
date = 2026-10-09
+++

Most of my sessions touch more than one repository. An agent project needs a
fix in the CLI it drives; a session in some project notices something to fix
in my NixOS dotfiles. Each of those repositories carries its own `CLAUDE.md`,
`.claude/rules/`, skills and hooks: the brief that makes an agent behave
there. Edit the neighbour from the wrong session, and the agent works without
any of it.

## What Claude Code loads, and from where

A session's context is anchored to the directory it starts in:

- **At launch:** the `CLAUDE.md` of the start directory and of every parent
  above it.
- **On demand:** `CLAUDE.md` files, rules and skills in subdirectories of the
  start directory, the first time the agent reads a file there.
- **Everywhere else:** nothing.

So when a session in `agent/` edits `../cli/src/main.rs`, it does so with none
of `cli/`'s instructions loaded. No error, no warning. The edit ignores the
conventions, the test commands and the "never do X" lines that repository
wrote down for exactly this agent.

## The native workarounds, and where they stop

| Approach                                | `CLAUDE.md` | Nested `CLAUDE.md` | Skills    | Hooks |
|-----------------------------------------|-------------|--------------------|-----------|-------|
| Edit `../cli` directly                  | –           | –                  | –         | –     |
| `permissions.additionalDirectories`     | –           | –                  | –         | –     |
| `--add-dir ../cli` with the env var     | ✓           | –                  | ✓         | –     |
| Start in a common parent directory      | on demand   | on demand          | on demand | –     |
| A subagent                              | –           | –                  | –         | –     |
| **A session started in `../cli`**       | ✓           | on demand          | ✓         | ✓     |

**`--add-dir`** comes closest. With `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`
set, it loads the added repository's root `CLAUDE.md` and its skills. But the
directories are chosen at launch, so a session that *might* touch the CLI
carries the CLI's whole brief from the first prompt, even on a day I only
wanted to work in the agent project. Nested `CLAUDE.md` files below the added
root never load, and that repository's hooks never run.

**`permissions.additionalDirectories`** in `settings.json` grants file access
and loads no instructions at all.

**Starting in a common parent** picks up nested files on demand, but each
repository's own settings and hooks stay unloaded, and the session no longer
belongs to any one project.

**A subagent** can't help either: the Agent tool takes no working directory.
A subagent starts in its parent's project with the same blind spot, and its
worktree isolation is a copy of the *parent's* repository.

What's missing is a way to say "this change belongs to that repository, so
load it as you would there", decided at the moment the change comes up.

## Hand the change to a session that lives there

Claude Code can already start a session anywhere, in the background:

```sh
cd ../cli && claude --bg --worktree cli-exit-codes --name cli-exit-codes \
  --permission-mode auto "<task>"
```

That is a full session rooted in `../cli`. It loads that repository's
`CLAUDE.md`, its nested files as it reads them, its rules, skills and hooks:
everything you'd get by opening a terminal there. `--worktree` gives it its
own checkout on the branch `worktree-cli-exit-codes`, so it never collides
with whatever is open in the main checkout. When it's done, it reports back
with `SendMessage`, and the report lands in the session that started it.
`claude agents`, `attach`, `logs`, `stop` and `rm` manage it from a terminal.

Nothing loads until a change needs that repository, and the session doing the
work is the one that pays for its context.

## The setup: one rule file

No wrapper, no tooling. One global rule in
`~/.claude/rules/other-repositories.md` teaches every session the move:

```markdown
# Changes to other repositories

Never change a git repository other than the session's project from this
session, and never through a subagent: only a session started in that
repository loads its CLAUDE.md files, hooks and skills. Start a fresh
background session there, in a worktree of its own:

    cd <repo> && claude --bg --worktree <name> --name <name> --permission-mode <this session's mode> "<task>"

- The task stands alone: what to change and why, how to verify it, commit on
  its branch, and report back with SendMessage to this session's name
  (ListAgents shows it).
- When it reports, check its branch, then merge it the same way: a fresh
  background session in that repository, told to merge `worktree-<name>`
  into the branch it belongs on and finish as that repository's rules say.
  Nobody switches the main checkout's branch. Then `claude stop` and
  `claude rm` the first session.
- Never message a session already running in that repository.
- A subagent never starts one: it stops and tells the main agent.
- Shell changes count too (`sed -i`, a formatter, `git commit`). Reading
  another repository is fine.
```

A few of those lines carry most of the weight:

- **Fresh, never an existing session.** Another session in that repository
  may be deep in something unrelated, with a context full of it. A new one
  starts clean.
- **The task stands alone.** The new session knows nothing of yours, so it has
  to be told what to change, why, and how to check it. That makes a better
  hand-over than "fix it like we discussed".
- **The permission mode travels along.** A subagent inherits its parent's
  mode; a background session gets it from `--permission-mode`.
- **Merging is part of the loop.** Check the branch the session reports, then
  let another fresh session in that repository merge it and finish the way
  that repository says: formatting, tests, a rebuild.

One prerequisite: Claude Code won't start a background session in a folder it
doesn't trust yet, so open `claude` in each repository once.

## Why this beats stretching one session

A repository's `CLAUDE.md` is its brief for an agent. Every native workaround
tries to carry that brief into a session that started somewhere else, and each
one drops part of it. Starting the session where the code lives drops
nothing, costs nothing until it's needed, and leaves each repository in charge
of how it's changed.

*Measured on Claude Code 2.1.292.*
