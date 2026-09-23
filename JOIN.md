# Joining Group-2

**Who this is for:** a coding agent (Claude Code, Cursor, Codex, …) working for a member of
TECH-UB 24 section 002, Group 2 (Brightspace: "PiPAI Fall 2026 002 2"). Humans can follow it too.

**How to use it:** go top to bottom and don't skip a step's check. Steps marked **[HUMAN]** are for
the person, not the agent: signing in, accepting invites, anything with a password or token. When
you reach one, stop and ask.

Repo: https://github.com/rongsibei-spec/Group-2 (public)

## The requirement

Brightspace item **"Team Github repo"**, due **Thu Sep 24, 2026, 12:00 PM (New York time)**:
**every member has at least one pull request, opened from their own GitHub account and merged into
`main`.** An open or approved PR doesn't count; it has to be merged. Late work is not accepted.

Team convention: your PR puts your first name into `team_details.py` in place of one `nameN`
placeholder.

## 0. Access  **[HUMAN]**

1. Accept the collaborator invite from `rongsibei-spec` (email, or
   https://github.com/rongsibei-spec/Group-2/invitations).
2. Sign in the GitHub CLI: `gh auth login` (GitHub.com → HTTPS → web browser). No `gh`? Install it
   from https://cli.github.com.

**Check:**

```bash
gh repo view rongsibei-spec/Group-2 --json viewerPermission -q .viewerPermission
```

It should print `WRITE`, `MAINTAIN` or `ADMIN`. Anything else, or "Could not resolve", means the
invite isn't accepted or `gh` is signed in as someone else. Stop and tell your human.

## 1. Clone and set your identity

```bash
gh repo clone rongsibei-spec/Group-2
cd Group-2
LOGIN=$(gh api user -q .login)
git config user.name "$(gh api user -q '.name // .login')"
git config user.email "$(gh api user -q .id)+$LOGIN@users.noreply.github.com"
```

This sets repo-local config only, not `--global`. The noreply address makes the commit show as
yours on GitHub.

## 2. Pick your placeholder

Everyone edits the same line, so first look at what's already taken, both on `main` and in open
PRs:

```bash
git fetch origin && git show origin/main:team_details.py
gh pr list --repo rongsibei-spec/Group-2 --state open --json number,author,title
gh pr diff <number> --repo rongsibei-spec/Group-2      # for each open PR
```

Take the **lowest-numbered `nameN` that is still a placeholder on `main` and that no open PR
replaces.** Use your human's first name, and ask them if you aren't sure.

## 3. Make the PR

```bash
git switch -c <firstname>-branch origin/main       # team convention, e.g. rodrigo-branch
# edit team_details.py: replace ONLY your nameN with your first name; keep everything else byte-for-byte
python3 team_details.py                            # must still run and print the line with your name
git diff --name-only                               # must print only team_details.py
git commit -am "Add <FirstName> to team details"
git push -u origin <firstname>-branch
gh pr create --repo rongsibei-spec/Group-2 --base main \
  --title "Add <FirstName> to team details" \
  --body "Replaces <nameN> with <FirstName> in team_details.py. Checked: \`python3 team_details.py\` prints the updated line."
```

**Check:** `gh pr view --json url,files -q '.url, .files[].path'` shows your PR and only
`team_details.py`.

## 4. If your PR shows a conflict

This happens whenever someone else's name PR merges before yours: two edits to the same line. For
a one-line change, the simplest fix is to redo the edit on top of the latest `main`:

```bash
git fetch origin
git switch -C <firstname>-branch origin/main       # start over from current main
# re-apply your single edit (main now has the other names filled in; keep them)
python3 team_details.py
git commit -am "Add <FirstName> to team details"
git push --force-with-lease -u origin <firstname>-branch   # OK on your own branch, never on main
```

The open PR updates itself. You don't need a new one.

## 5. Get it merged

Tell the group chat your PR is up. Once a teammate has had a look, merge it:

```bash
gh pr merge <number> --repo rongsibei-spec/Group-2 --squash --delete-branch
```

If it hasn't merged by **Wed Sep 23 evening**, ping the group so there's time before the Thursday
noon deadline. Don't merge other people's PRs without telling them.

## 6. Confirm

```bash
gh pr list --repo rongsibei-spec/Group-2 --state merged --author "@me" --json number,title,mergedAt
```

One row or more means you're done. Empty right after merging? GitHub's search index lags by up to a
minute, so wait and run it again.

Whole-group check: every member's login should appear.

```bash
gh pr list --repo rongsibei-spec/Group-2 --state merged --limit 100 --json author -q '[.[].author.login] | unique | .[]'
```

## Rules

- Never push to `main` directly. Always use a branch and a PR; `main` isn't protected, so this is
  on us.
- Touch only your own placeholder, and never change someone else's name.
- No secrets, tokens or passwords in commits. The repo is **public**.
- Anything that needs a login, an invite, or a judgement call about the team goes to your human.
