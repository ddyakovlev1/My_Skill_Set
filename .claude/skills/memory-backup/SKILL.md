---
name: "memory-backup"
description: "Back up all of Claude's memory files about the user into a private GitHub repo. Use when asked to back up memory or when a scheduled backup task runs."
---

# Memory backup

Copy every file from Claude's memory store into a GitHub repository, commit, and get the changes onto the default branch (usually `main`) without any manual step. The repo is a mirror: after the run, its `memory/` folder on the default branch matches memory exactly. Git history keeps older versions.

This skill is READ-ONLY for memory. Never write, edit or delete memory files during a backup.

The run must end with the changes merged into the default branch. Never leave an open branch or pull request for the user to merge by hand.

## Input

The target repo comes from the skill args or the task prompt, as `owner/repo`. If no repo is given, stop and report that the repo name is missing. Do not guess.

## Steps

1. **Collect the file list.** Call `mcp__memory__memory_list` with no prefix. If the result has more entries (it is capped), call again with `cursor` set to the last path until you have every path.

2. **Read every file.** Call `mcp__memory__memory_read` with lists of up to 20 paths per call. Keep each file's content exactly as stored (frontmatter included). Drop only tool wrapper text that is not file content, such as version tokens or size notes.

3. **Attach the repo.** Call `mcp__claude-code-remote__add_repo` with the owner, repo and `access: "push"`. If it returns an access error, stop and report its exact reason. Then clone with the command the tool returns. Find the default branch (`git remote show origin` or `gh repo view --json defaultBranchRef`) and check it out. Run `git pull` so you start from the latest state.

4. **Check the repo is private.** If you can check visibility (for example `gh repo view --json visibility` or the add_repo result), and the repo is PUBLIC, stop without pushing and report it. Memory holds personal data.

5. **Write the mirror.** In the clone, on the default branch:
   - Delete the old `memory/` folder.
   - Write each memory file to `memory/<same path>` (for example `/topics/pc.md` → `memory/topics/pc.md`).
   - Write `memory/INDEX.md`: backup date and time (Europe/Moscow), number of files, and a list of paths with their one-line descriptions from frontmatter.
   - Do not touch other files in the repo (README, etc.).

6. **Commit.**
   - Run `git add -A`.
   - If nothing changed except the timestamp in INDEX.md, discard changes (`git checkout -- .`), report "no changes since last backup" and stop.
   - Otherwise commit with message `Memory backup YYYY-MM-DD`.

7. **Get it onto the default branch.** Try in this order and stop at the first that works:
   1. `git push origin <default-branch>`.
   2. If the push is rejected because the remote moved: `git pull --rebase origin <default-branch>`, then push again. If the rebase conflicts inside `memory/`, keep your version (fresh memory is the truth): `git checkout --theirs memory/ && git add memory/ && git rebase --continue` (during a rebase, "theirs" is your commit). Never resolve by taking the remote version of `memory/`.
   3. If direct push to the default branch is not allowed (protected branch, or the session may only push to its own branch): push the commit to a new branch `memory-backup/YYYY-MM-DD` (or the branch the session allows), open a pull request into the default branch (`gh pr create --fill`), merge it (`gh pr merge --squash --delete-branch`; if the merge needs checks, use `--auto` and wait up to ~10 minutes, checking `gh pr view --json state`), then confirm the PR state is MERGED.
   - If none of these work, report the exact error and the branch / PR link, and say clearly that the merge did not happen.

8. **Verify.** Run `git fetch origin` and confirm `origin/<default-branch>` contains the backup commit (or the squash-merge commit).

9. **Report.** Send a short message in Russian with `SendUserMessage`: number of files, which files were added / changed / removed since the last backup (from `git diff --stat` of the commit), how it landed (direct push or merged PR), and the commit link. In an unattended run this message is the only thing the user sees, so always send it, also on failure.

## Failure rules

- If memory tools are not available in this session, report that clearly and stop.
- If the memory list is empty, do NOT push an empty mirror (it would wipe the backup). Report and stop.
- If push or merge fails, report the git or gh error text.