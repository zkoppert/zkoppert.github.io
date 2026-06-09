---
title: "When 'Always' Isn't Enough: Hard Guardrails for Copilot CLI"
date: 2026-06-08
categories: [Developer Tooling, GitHub Copilot]
tags: [github-copilot, copilot-cli, dotfiles, developer-experience, ai-agents]
image:
  path: /assets/img/posts/bouncercat.png
  alt: "Bouncer Octocat from the GitHub Octodex, standing guard"
---

## TL;DR

I have a rule in my Copilot CLI instructions: "always run a multi-model code review before opening a PR." I wrote it down. I told the agent. I watched the agent skip it. More than once.

The apologies were dramatic. "You're absolutely right, I should have run the multi-model review first. I won't skip it again." I have a toddler. I have heard a version of this speech before. The aspirations are real; the system that would back them up didn't exist.

For most rules, instructions are enough. For the rules I care most about, I needed guardrails. This post is about one I built: a PATH shim called `gh-guard` that turns "always" from a prompt into a gate.

![Animated GIF of the Mandalorian with the caption "This is the way"](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExNmJ6cWRvanYzZjg2cWo4c2x4NDAzcHd5b3Z0d2xzcjhlazNxa3l0NyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/1hAxQTH0HEWS3L0oRF/giphy.gif)

## The gap between instructions and behavior

My `copilot-instructions.md` is around 40KB. Most of it is faithfully followed - tone preferences, naming conventions, PR description structure. The rules that lose are a specific shape: they have an external side effect (notifying reviewers, opening a PR), they sit between the agent and "task complete," and the agent has a natural incentive to call the work done.

A typical failure looks like this. I ask Copilot to fix a bug. It investigates, writes a fix, runs the tests, and confidently says "ready to open a PR." The instruction file is somewhere in context, but "always run a multi-model review" is one line out of hundreds. The agent does what feels like the natural next step, and the rule loses.

I tried the obvious prompt-engineering moves first: pinning the rule near the top of the file, repeating it in two places, restructuring it as a numbered workflow. Each one helped. None of them gave a guarantee. Prompts are probabilistic; the context window is finite; attention is uneven. For binary, high-consequence rules, "99% of the time" still isn't the answer I want.

If a rule actually matters that much, it shouldn't live only in a prompt. It should live in a gate.

## The shim trick

Here is the whole technique:

1. Write a script with the same name as the command you want to intercept.
2. Put it in a directory that comes earlier on `$PATH` than the real binary.
3. The script inspects what was asked, decides whether to allow it, and then calls the real binary.

That is it. The shell searches `$PATH` left to right and runs the first match. Tools like [rbenv](https://github.com/rbenv/rbenv#understanding-shims), pyenv, and asdf have used this pattern for years to switch language runtimes. I'm using it to enforce my own rules on a Copilot CLI agent that I cannot fully trust to follow them on its own.

## Implementing gh-guard

[`gh-guard`](https://github.com/zkoppert/dotfiles/blob/main/bin/gh-guard) is a ~280-line bash script that wraps the `gh` CLI. It guards two specific subcommands:

- **`gh pr ready`** marks a draft PR as ready for review, which notifies all reviewers. My rule is to never do this without my explicit ask, because the agent should never decide on its own to interrupt other humans. (`gh pr ready --undo` reverts a PR back to draft and does not notify anyone, so it passes through without gating.)
- **`gh pr create`** opens a new PR. My rule is to always complete a multi-model code review first (at least three review agents, different models, synthesize findings, address verified issues).

Without enforcement, both rules got skipped. The behavior with `gh-guard` installed depends on who is calling:

**From a non-interactive shell (agent invocations):**

- `gh pr ready` is blocked unless `ZACK_CONFIRMED_PR_READY=1` is set in the same invocation.
- `gh pr create` is blocked unless **both** a multi-model review summary (>=200 bytes) already exists at `<git-dir>/copilot-pr-review/<branch>.md` **and** `ZACK_CONFIRMED_PR_CREATE=1` is set on the `gh pr create` invocation. (If the shim can't resolve the repo or branch - detached HEAD, no checkout - the marker path can't be derived, so the env var alone is accepted as a fallback.)

**From an interactive terminal (me, typing):**

- The shim prints what's about to run and prompts `Continue? [y/N]`. For `gh pr create` it also reports whether the review marker exists. It's a checkpoint, not a wall. I'm present, I can make the call.

Everything else passes through to the real `gh` unchanged. It's a checkpoint on two specific subcommands, not a tax on every `gh` command.

The gate verifies a *deliberate action*, not that I actually ran three review agents and addressed every finding. A 200-byte garbage file plus the env var would satisfy it. That's intentional: I want a speed bump for absent-mindedness, not a fortress against a determined adversary.

## How it works

Three pieces matter.

**Finding the real binary without infinite recursion.** The shim has to call the actual `gh`, not itself. It walks `$PATH`, skips both the directory it was invoked from *and* the directory the symlink resolves to, and picks the first `gh` it finds.

```bash
SCRIPT_PATH="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
SELF_REAL_PATH="$(cd "$(dirname "$(readlink "$0" 2>/dev/null || echo "$0")")" && pwd)"
REAL_GH=""
IFS=':' read -ra PATH_ENTRIES <<< "$PATH"
for entry in "${PATH_ENTRIES[@]}"; do
  candidate="$entry/gh"
  [ ! -x "$candidate" ] && continue
  resolved_dir="$(cd "$entry" 2>/dev/null && pwd || echo "$entry")"
  if [ "$resolved_dir" = "$SCRIPT_PATH" ] || [ "$resolved_dir" = "$SELF_REAL_PATH" ]; then
    continue
  fi
  REAL_GH="$candidate"
  break
done
```

Resolving both paths is the difference between "works as long as nobody puts the dotfiles `bin/` on PATH" and "works no matter where you put it."

**Parsing past gh's global flags.** `gh -R owner/repo pr ready` puts a flag before the subcommand, so a naive "is `$1` equal to `pr`?" check misses it. The shim walks the arguments, consumes known global flags and their values, and grabs the first two positional tokens as the subcommand and operation.

**Differentiating interactive shells from automation.** If I'm running `gh pr ready` from my own terminal, I want a yes/no prompt. If an agent is running it from a non-interactive shell, I want a hard refusal unless the env var is set. `[ -t 0 ] && [ -t 1 ]` is a good-enough heuristic for my agent-vs-human split. (Some automation can allocate a TTY; some human flows are non-interactive. For my actual setup, it's reliable.)

On pass, the script just calls `exec "$REAL_GH" "$@"` and the shim is invisible.

## What I learned the hard way

A few details that took longer than they should have:

- **Worktrees break naive paths.** Inside a worktree, `<repo>/.git` is a small text file pointing elsewhere, not a directory. `git rev-parse --absolute-git-dir` returns the per-worktree git directory correctly, so the marker file lives in the right place whether you are in the main checkout or a worktree.
- **Branch names need encoding.** `feat/foo` would collide with anything named `feat%2Ffoo` if you used the raw branch name as a filename. I percent-encode `%` and `/` so most branch names get distinct markers. (On a case-insensitive filesystem like macOS default, branches that differ only in case will still collide. I haven't hit that in practice; I'd add a hash if I did.)
- **Empty markers must fail.** A bare `touch` should not satisfy the guard. I require the marker to be at least 200 bytes, which is enough to fit a real (even if terse) synthesized review.
- **Help flags should pass through.** `gh pr create --help` is a legitimate introspection call. The shim scans for `--help`, `-h`, or `help` anywhere in the arguments and falls through without gating.
- **Invoke through the PATH shim, not the real script.** If the shim is called by its actual path, it can find itself again in PATH lookup and fork-recurse. Install it at a stable PATH location (the install instructions drop it at `~/.local/bin/gh`) and let the shell resolve it as `gh`.

## What about Copilot hooks?

Copilot CLI has a [hooks system](https://docs.github.com/en/copilot/concepts/agents/hooks) that can intercept tool calls before they execute and approve or deny them. If your problem is purely Copilot-shaped, hooks are the more native answer.

I went with a PATH shim because the agent isn't the only one who skips steps. I do too. It's 11pm, I just rebased, muscle memory takes over, and I'm typing `gh pr ready` before my brain catches up. Hooks fire when Copilot is driving. The shim fires whenever the command runs - Copilot, Claude Code, Cursor, an ad-hoc shell script, or me at the keyboard. Same rule, same gate, regardless of who typed it.

For me, that's the real value. The gate doesn't care whether the mistake is mine or the model's. It just stops the mistake.

Both can coexist. If you're a Copilot-only shop and you never want a prompt interrupting your own typing, start with hooks. If you want one rule that holds whether the human or the agent is driving, the shim travels further.

## The limits

This is a convention enforcer, not an adversarial gate. A determined agent has a few bypass paths:

- `gh api` calls, whether REST (`gh api repos/:owner/:repo/pulls`) or GraphQL (a `createPullRequest` mutation), are not gated. Blocking all of `gh api` would break too much legitimate read traffic, so I accept the gap.
- `gh alias set prc 'pr create'` followed by `gh prc`. Aliases resolve inside the real `gh` after the shim has already dispatched, so they bypass the matcher. I just don't alias the guarded subcommands.
- Calling the real `gh` by absolute path (`/opt/homebrew/bin/gh pr create ...`) skips the shim entirely. Any agent that knows where the real binary lives can route around it.
- Setting the env var and writing a 200-byte garbage file satisfies the marker check. The goal is to force a deliberate action, not to be unbypassable.

If you need a true adversarial gate, the right answer is server-side: branch protection rules, required status checks, and a CODEOWNERS file. Shims are a *local* guard against your own automation cutting corners.

![Animated GIF of Gandalf raising his staff in Moria and shouting "You shall not pass!" from The Fellowship of the Ring](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExaXZtd2hrZ29tNHZsdDdnOW9razRhNXByOTJsMGE5ajBibXAybDd4ayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YkfhemFXalh7O/giphy.gif)

## Where else this pattern fits

A few other rules I have considered shimming:

| Command | What the shim could enforce |
|---|---|
| `rm -rf` | Refuse paths outside the current directory unless a confirmation env var is set |
| `git push --force` | Block against protected branches even if branch protection is off locally |
| `kubectl apply` | Log every invocation to a daily audit file with the resolved cluster context |
| `aws s3 rb --force` | Refuse unless the bucket name matches a known disposable prefix |
| `npm publish` | Refuse unless `CHANGELOG.md` was touched in the last commit |

The common shape: rules where the command surface is narrow (one or two subcommands), the outcome is externally observable (a PR exists, a bucket is gone, a package is on the registry), and the cost of one skipped check is meaningfully larger than the cost of one extra confirmation. Those are the rules that earn a shim. Everything else stays in the prompt.

## Install it

If you want to try the pattern, the install is short. You don't need to clone my dotfiles - grab the script straight from the source:

```bash
# Pull the shim into a directory you control. Read it first.
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/zkoppert/dotfiles/main/bin/gh-guard \
  -o ~/.local/bin/gh
chmod +x ~/.local/bin/gh

# Make sure ~/.local/bin comes before wherever the real gh lives.
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

Open a new shell and verify the shim is in front:

```bash
$ which -a gh
/Users/you/.local/bin/gh         # shim
/opt/homebrew/bin/gh             # real gh
```

If the shim isn't first, your `PATH` export landed in the wrong rc file or your shell hasn't reloaded.

## Consider building your own shim

`gh-guard` is one rule for one command. The interesting part is the pattern, not the specific gate. If you have an "always" or "never" rule that you keep restating to Copilot, or that lives in a `CONTRIBUTING.md` you wish people read more carefully, that rule is probably a shim waiting to happen.

The recipe is small:

1. Pick the command and the subcommand you want to gate.
2. Drop a script with the command's name into a directory on `PATH` ahead of the real binary.
3. Inside the script, find the real binary by walking `PATH` and skipping your own directory.
4. Intercept the verb you care about; fall through to the real binary for everything else.
5. Add an explicit escape hatch (a flag, an env var, or a confirmation prompt) so you can ship when you mean to.
