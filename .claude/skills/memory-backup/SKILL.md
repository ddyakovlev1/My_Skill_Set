---
name: "memory-backup"
description: "Back up all of Claude's memory files about the user into a private GitHub repo (current copy on main + dated snapshots), or restore memory from it. Use when asked to back up or restore memory, or when a scheduled backup task runs."
---

# Memory backup

Copy every file from Claude's memory store into a GitHub repository. The repo keeps two things:

- **Current copy** — the `memory/` folder on the default branch (usually `main`). After the run it matches memory exactly. This is the copy to restore from when needed.
- **Dated snapshots** — the branch `snapshots`. It has one folder per backup date, named `DD.MM.YY` (Europe/Moscow date, for example `07.10.26`). Each folder is a full frozen copy of memory on that date. Old snapshot folders are never changed or deleted.

This skill is READ-ONLY for memory. Never write, edit or delete memory files during a backup (restore is the one exception, see below).

The run must end with the changes on the default branch and on `snapshots`. Never leave an open branch or pull request for the user to merge by hand.

## Input

The target repo comes from the skill args or the task prompt, as `owner/repo`. If no repo is given, stop and report that the repo name is missing. Do not guess.

## Steps

1. **Collect the file list.** Call `mcp__memory__memory_list` with no prefix. If the result has more entries (it is capped), call again with `cursor` set to the last path until you have every path.

2. **Read every file.** Call `mcp__memory__memory_read` with lists of up to 20 paths per call. Keep each file's content exactly as stored (frontmatter included). Drop only tool wrapper text that is not file content, such as version tokens or size notes.

3. **Attach the repo.** Call `mcp__claude-code-remote__add_repo` with the owner, repo and `access: "push"`. If it returns an access error, stop and report its exact reason. Then clone with the command the tool returns. Find the default branch (`git remote show origin` or `gh repo view --json defaultBranchRef`) and check it out. Run `git pull` so you start from the latest state.

4. **Check the repo is private.** If you can check visibility (for example `gh repo view --json visibility` or the add_repo result), and the repo is PUBLIC, stop without pushing and report it. Memory holds personal data.

5. **Get today's date.** Call `mcp__claude_ai__current_time` (or use `TZ=Europe/Moscow date`). Build `DATE=DD.MM.YY` (for example `07.10.26`) and `ISO=YYYY-MM-DD`.

6. **Update the current copy.** In the clone, on the default branch:
   - Delete the old `memory/` folder.
   - Write each memory file to `memory/<same path>` (for example `/topics/pc.md` → `memory/topics/pc.md`).
   - Write `memory/INDEX.md`: backup date and time (Europe/Moscow), number of files, and a list of paths with their one-line descriptions from frontmatter.
   - Do not touch other files in the repo (README, etc.).
   - Run `git add -A`. If nothing changed except the timestamp in INDEX.md, discard changes (`git checkout -- .`), report "no changes since last backup" and stop. Do not make a snapshot in this case.
   - Otherwise commit with message `Memory backup <ISO>`.

7. **Push the current copy to the default branch.** Try in this order and stop at the first that works:
   1. `git push origin <default-branch>`.
   2. If the push is rejected because the remote moved: `git pull --rebase origin <default-branch>`, then push again. If the rebase conflicts inside `memory/`, keep your version (fresh memory is the truth): `git checkout --theirs memory/ && git add memory/ && git rebase --continue` (during a rebase, "theirs" is your commit). Never resolve by taking the remote version of `memory/`.
   3. If direct push to the default branch is not allowed (protected branch, or the session may only push to its own branch): push the commit to a new branch `memory-backup/<ISO>` (or the branch the session allows), open a pull request into the default branch (`gh pr create --fill`), merge it (`gh pr merge --squash --delete-branch`; if the merge needs checks, use `--auto` and wait up to ~10 minutes, checking `gh pr view --json state`), then confirm the PR state is MERGED.
   - If none of these work, report the exact error and the branch / PR link, and say clearly that the merge did not happen. Still try step 8.

8. **Add the dated snapshot.** Work in a separate folder so the default branch stays untouched:
   - If `origin/snapshots` exists: `git fetch origin snapshots && git worktree add ../snap origin/snapshots && cd ../snap && git switch -c snapshots-tmp`.
   - If it does not exist: `git worktree add --orphan -b snapshots-tmp ../snap && cd ../snap`, and write a short `README.md`: "Dated snapshots of Claude memory, one folder per backup date (DD.MM.YY). The current copy for restore is `memory/` on the default branch."
   - Copy the fresh `memory/` folder from the main clone into `<DATE>/` (so the files land at `<DATE>/topics/pc.md` and so on, plus `<DATE>/INDEX.md`). If `<DATE>/` already exists (second backup on the same day), replace only that folder. Never change other date folders.
   - Commit with message `Snapshot <DATE>` and run `git push origin HEAD:snapshots`. If rejected because the remote moved, `git pull --rebase origin snapshots` and push again.
   - Remove the worktree when done (`cd - && git worktree remove ../snap`).

9. **Verify.** Run `git fetch origin` and confirm `origin/<default-branch>` contains the backup commit (or the squash-merge commit) and `origin/snapshots` contains the folder `<DATE>/`.

10. **Report.** Send a short message in Russian with `SendUserMessage`: number of files, which files were added / changed / removed since the last backup (from `git diff --stat` of the commit), how the current copy landed (direct push or merged PR), the name of the snapshot folder, and links to both commits. In an unattended run this message is the only thing the user sees, so always send it, also on failure.

## Restore

If the user asks to restore memory from the backup: take the files from `memory/` on the default branch (or from the snapshot folder `<DATE>/` on `snapshots`, if the user names a date). Show the user what will change and get an explicit confirmation before writing anything to memory. Restore is the only case where this skill writes memory, and only after that confirmation.

## Failure rules

- If memory tools are not available in this session, report that clearly and stop.
- If the memory list is empty, do NOT push an empty mirror or an empty snapshot (it would wipe the backup). Report and stop.
- If push or merge fails, report the git or gh error text.
