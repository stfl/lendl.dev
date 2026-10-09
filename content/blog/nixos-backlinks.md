+++
title = "Backlinks: let a tool write its own config and keep it in the flake"
description = "A small home-manager option that links a tool's config from $HOME back into the dotfiles checkout, so what Claude Code writes to ~/.claude/ lands in git, with no write ever hitting the read-only Nix store."
date = 2026-04-18
updated = 2026-09-17
+++

Declarative configuration and self-configuring tools pull in opposite
directions. NixOS wants every file under `$HOME` to be the output of an
evaluation, read-only, reproducible on the next machine. Claude Code wants to
write `~/.claude/settings.json` when I run `/config`, drop a plugin into
`~/.claude/skills/`, and add a rule when I ask it to. One of them has to give.

My dotfiles are a flake; the `~/.claude/` tree is managed by home-manager. The
module I use to make both sides happy is called `backlinks`. The idea: the file
lives at the path the tool expects and writes to, and that path is a symlink
back into the git checkout of the flake. Edits made by the tool land in the
working tree, show up in `git status`, get reviewed and committed, and reach
the next machine through the same repo as everything else.

## Three ways that don't work

**`home.file` / `xdg.configFile`.** The standard route copies the file into the
Nix store and links `~/.claude/settings.json` at it. The store is read-only,
so the first `/config` save fails with `EROFS`. Fine for a file the tool only
reads; useless for one it owns.

**`mkOutOfStoreSymlink` by hand.** home-manager can link at an absolute path
outside the store. That is the right primitive, and `backlinks` is built on
it, but used directly it leaves gaps: every file needs its own entry with a
hand-written absolute path, the formatter has no idea the file is
service-owned and reformats it under the tool's feet, and the link still
routes through two symlinks inside the store, which breaks a tool that saves
by resolving one hop and renaming a temp file beside the result. Claude Code
saves `settings.json` exactly that way.

**Leave the files out of Nix.** The tool writes happily, and nothing is
versioned. The config that makes the agent behave well, the rules, the
skills I asked it to write, exist on one machine only, and a reinstall starts
from zero.

| | Tool can write | Versioned in git | Reproduced on next machine |
|---|---|---|---|
| `home.file` into the store | no | yes | yes |
| `mkOutOfStoreSymlink` by hand | mostly | yes | yes, with manual bookkeeping |
| Unmanaged | yes | no | no |
| `backlinks` | yes | yes | yes |

## The option

`backlinks` is a single home-manager module, registered once in
`home-manager.sharedModules`, so any home module can declare a link without
importing anything. A key is a `$HOME`-relative target; a value is a path
inside the flake, or an attrset of name to path for one link per entry under
the target:

```nix
backlinks = {
  ".claude/settings.json" = myLib.backlink.mkDirect ../../config/claude/settings.json;
  ".claude/agents" = ../../config/claude/agents;
  ".claude/commands" = ../../config/claude/commands;
  ".claude/rules" = myLib.backlink.allMds ../../config/claude/rules;
  ".claude/skills" =
    {
      claude-self-config = ../../config/agents/skills/claude-self-config;
      rust = ../../config/agents/skills/rust;
    }
    // myLib.backlink.allDirs ../../config/claude/skills;
};
```

The hard part is the one thing Nix cannot tell you: where the checkout is.
Inside an evaluation, the flake's own source is a store path with a hash in
its name. So the module pins the location by convention, every host checks the
flake out at `$XDG_CONFIG_HOME/dotfiles`, and computes the repo-relative part
from the Nix path:

```nix
relativeToFlake = path: let
  flakeRootStr = toString ./..;
  prefix = flakeRootStr + "/";
  pathStr = toString path;
in
  assert lib.assertMsg (lib.hasPrefix prefix pathStr)
  "myLib.relativeToFlake: ${pathStr} is not inside the flake root ${flakeRootStr}.";
    lib.removePrefix prefix pathStr;

checkoutPath = config: path: "${config.xdg.configHome}/dotfiles/${relativeToFlake path}";

mk = config: path: config.lib.file.mkOutOfStoreSymlink (checkoutPath config path);
```

The module flattens attrset values into one entry per name and turns the
result into `home.file`:

```nix
flat =
  lib.concatMapAttrs (
    target: value:
      if lib.isAttrs value && !isDirect value
      then lib.mapAttrs' (name: lib.nameValuePair "${target}/${name}") value
      else {${target} = value;}
  )
  cfg;

home.file = lib.mapAttrs (_: path: {source = myLib.backlink.mk config path;}) viaStore;
```

`allDirs` and `allMds` read a repo directory at evaluation time and expand it
into that attrset shape. A new `config/claude/rules/foo.md` or a new directory
under `config/claude/skills/` is picked up on the next rebuild with no Nix
edit. One link per entry rather than one link for the whole directory, because
`~/.claude/skills/` is a real directory that also receives read-only store
copies of pinned third-party skills; a wholesale symlink would collide, and
the collision is a home-manager evaluation error, which is the behaviour I
want.

## The one-hop link for atomic savers

On disk, a store-routed backlink is three hops:

```
~/.claude/agents
  -> /nix/store/…-home-manager-files/.claude/agents
  -> /nix/store/…-hm_agents
  -> /home/stefan/.config/dotfiles/config/claude/agents
```

Claude Code saves `settings.json` by resolving one symlink and renaming a temp
file next to the result. Through that chain the temp file is staged inside the
store, and the save fails with `EROFS`. `mkDirect` wraps a path so activation
links it straight at the checkout, bypassing `home.file`:

```nix
home.activation.directBacklinks = lib.hm.dag.entryAfter ["linkGeneration"] (''
  linkDirect() {
    local target="$HOME/$1" source="$2"
    if [[ -L $target && $(readlink "$target") == "$source" ]]; then
      return 0
    fi
    if [[ -e $target && ! -L $target ]]; then
      run $HOME_MANAGER_BACKUP_COMMAND "$target"
    fi
    run mkdir -p $VERBOSE_ARG "$(dirname "$target")"
    run ln -Tsf $VERBOSE_ARG "$source" "$target"
  }
'' + …);
```

`readlink ~/.claude/settings.json` prints the checkout path, and `/config`
persists. A plain file already in the way is handed to
`home-manager.backupCommand`, which moves it out of the tree rather than
leaving a `.bak` sibling that Claude Code would scan as a duplicate.

## The formatter has to know

The declared set is read back at the flake level and fed to treefmt as its
exclude list:

```nix
backlinkGlobs = myLib.backlink.globs (map
  (host: inputs.self.nixosConfigurations.${host}.config.home-manager.users.${USER}.backlinks)
  ["kondor" "pirol"]);
```

Declaring a backlink is what stops `nix fmt` from reformatting a file a
running service owns, and dropping the declaration re-includes it. This is
why a symlink made by hand with `mkOutOfStoreSymlink` is discouraged in the
repo: the formatter would not know to skip the file.

## End to end with Claude Code

I ask Claude Code to deny an MCP connector in every session. It edits
`config/claude/settings.json` in the dotfiles checkout, because its Edit tool
refuses to write through a symlink and a skill in the repo tells it that the
repo path is the one to edit. The same bytes are what `~/.claude/settings.json`
resolves to, so the running session sees them. `git -C ~/.config/dotfiles
diff` shows the change:

```diff
+  "deniedMcpServers": [
+    {
+      "serverName": "claude.ai Gmail"
+    }
+  ],
```

I read it, commit it with a message that says why, and push. On the laptop,
`git pull` is the deployment: the link already points at the checkout, so the
new content is live without a rebuild. A rebuild is needed only when a *new*
link has to exist, say a new skill directory. For that there is a bootstrap
flow: create the directory in the repo, `ln -s` it into `~/.claude/skills/` by
hand so the current session sees it, `git add` it so the flake sees it, and
let the next `home-manager switch` replace the manual link with its own,
pointing at the same path.

## Where it bends

**The closure does not contain the config.** A generation points at a
working tree, which can be dirty or on any commit. Reproducibility moves from
"the store has it" to "git has it", which is the trade I am making on purpose,
but a rollback of the system generation does not roll back these files.

**Machines diverge until I commit.** Two hosts each writing their own
`settings.json` produce a merge conflict on pull. In practice one machine is
where the agent configures itself and the others follow, and the diff is
small enough to read.

**Tools that rewrite the whole file own its formatting.** The treefmt
exclusion keeps the formatter and the tool from fighting, at the cost of the
file looking however the tool likes to write it.

**Secrets.** A tool that writes a token into its own config would write it into
git. Claude Code keeps credentials in a separate file that is not linked;
anything that mixes secrets into a config file needs a different mechanism.

**Direct links are untracked.** home-manager does not know about a `mkDirect`
link, so removing the declaration leaves the symlink behind, dangling once the
repo file goes. And `programs.claude-code.settings` cannot be used alongside
it: it would generate `settings.json` into the store, and the direct link
overwrites that on activation.

**Only declared paths are versioned.** `~/.claude/` is a real directory; the
history, project memory, caches and plugin downloads the tool puts next to the
links stay local and unmanaged. That is the point: choose by who writes the
file and whether the result deserves a commit.

The module is one Nix file plus a handful of helpers; the snippets above are
most of it. What it buys is a tool that is allowed to configure itself, inside
a system that still has one source of truth.
