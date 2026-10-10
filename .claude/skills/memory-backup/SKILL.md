---
name: "memory-backup"
description: "Back up all of Claude's memory files about the user into a private GitHub repo (current copy + dated snapshots, all on main), or restore memory from it. Use when asked to back up or restore memory, or when a scheduled backup task runs."
---

# Memory backup

Copy every file from Claude's memory store into a GitHub repository. Everything lives on the default branch (usually `main`):

- **Current copy** — the `memory/` folder. After the run it matches memory exactly. This is the copy to restore from when needed.
- **Dated snapshots** — the `snapshots/` folder. It has one subfolder per backup date, named `DD.MM.YY` (Europe/Moscow date, for example `07.10.26`). Each subfolder is a full frozen copy of memory on that date. Old snapshot folders are never changed or deleted.

This skill is READ-ONLY for memory. Never write, edit or delete memory files during a backup (restore is the one exception, see below).

Use ONE branch only. Do one commit and push it directly to the default branch. Never create other branches or pull requests: nothing must be left for the user to merge by hand.

## Input

The target repo comes from the skill args or the task prompt, as `owner/repo`. If no repo is given, stop and report that the repo name is missing. Do not guess.

## Steps

1. **Collect the file list.** Call `mcp__memory__memory_list` with no prefix. If the result has more entries (it is capped), call again with `cursor` set to the last path until you have every path.

2. **Read every file.** Call `mcp__memory__memory_read` with lists of up to 20 paths per call. Keep each file's content exactly as stored (frontmatter included). Drop only tool wrapper text that is not file content, such as version tokens or size notes.

3. **Get the repo.** First try a plain `git clone https://github.com/<owner>/<repo>` (a scheduled task may already have access). If that fails, call `mcp__claude-code-remote__add_repo` with the owner, repo and `access: "push"` (load it with ToolSearch if needed), then clone with the command it returns. If there is still no access, stop and report the exact error. Check out the default branch and run `git pull`.

4. **Check the repo is private.** If you can check visibility (for example `gh api repos/<owner>/<repo> --jq .private`), and the repo is PUBLIC, stop without pushing and report it. Memory holds personal data.

5. **Get today's date.** Call `mcp__claude_ai__current_time` (or use `TZ=Europe/Moscow date`). Build `DATE=DD.MM.YY` (for example `07.10.26`).

6. **Update the current copy.** In the clone, on the default branch:
   - Delete the old `memory/` folder.
   - Write each memory file to `memory/<same path>` (for example `/topics/pc.md` → `memory/topics/pc.md`).
   - Write `memory/INDEX.md`: backup date and time (Europe/Moscow), number of files, and a list of paths with their one-line descriptions from frontmatter.
   - Do not touch other files in the repo (README, etc.).

7. **Check for changes.** Run `git add -A`. If nothing changed except the timestamp in INDEX.md, discard changes (`git checkout -- .`), report "no changes since last backup" and stop. Do not make a snapshot in this case.

8. **Add the dated snapshot.** Copy the fresh `memory/` folder to `snapshots/<DATE>/` (so the files land at `snapshots/<DATE>/topics/pc.md` and so on, plus `snapshots/<DATE>/INDEX.md`). If `snapshots/<DATE>/` already exists (second backup on the same day), replace only that folder. Never change other date folders.

9. **Commit and push.** `git add -A`, commit with a message in Russian, for example `Бэкап памяти <DATE>`, and run `git push origin <default-branch>`.
   - If the push is rejected because the remote moved: `git pull --rebase origin <default-branch>`, then push again. If the rebase conflicts inside `memory/`, keep your version (fresh memory is the truth): `git checkout --theirs memory/ && git add memory/ && git rebase --continue` (during a rebase, "theirs" is your commit). Never resolve by taking the remote version of `memory/`.
   - If direct push is not allowed, report the exact error. Do not open a pull request.

10. **Verify.** Run `git fetch origin` and confirm `origin/<default-branch>` contains the backup commit with `memory/` and `snapshots/<DATE>/`.

11. **Report.** Send a short message in Russian with `SendUserMessage`: number of files, which files were added / changed / removed since the last backup (from `git diff --stat` of the commit), the name of the snapshot folder, and a link to the commit. In an unattended run this message is the only thing the user sees, so always send it, also on failure.

## Restore

If the user asks to restore memory from the backup: take the files from `memory/` (or from `snapshots/<DATE>/`, if the user names a date). Show the user what will change and get an explicit confirmation before writing anything to memory. Restore is the only case where this skill writes memory, and only after that confirmation.

## Failure rules

- If memory tools are not available in this session, report that clearly and stop.
- If the memory list is empty, do NOT push an empty mirror or an empty snapshot (it would wipe the backup). Report and stop.
- If push fails, report the git error text.
