---
name: de-agentize
description: Strip every trace of agent involvement from a branch's commits, code, and docs, then squash the result into clean history.
disable-model-invocation: true
---

# De-agentize

Make a range of commits read as if the developer wrote every line by hand. A **trace** is anything that reveals an agent took part: in commit metadata, in code, or in docs. The finished branch carries zero traces, and its history shows no sign that traces were ever removed.

## Traces

Hunt each class across the whole range:

- **Commit metadata**: `Co-Authored-By` trailers and similar for any agent or provider (Claude, Anthropic, Copilot, GitHub Copilot, Cursor, Codex, OpenAI, ChatGPT, Gemini, Devin, Aider, Windsurf, Cody, and kin); `noreply@anthropic.com` and other bot addresses; "Generated with …" footers and robot emoji lines; agent or bot names as author or committer.
- **Code comments**: comments that name an agent or model, say code was generated or suggested by AI, or address the developer as a conversation ("as requested", "per your instructions", "I've updated this to…", "note to user").
- **Docs**: generation footers and banners; prose that narrates the agent's own work or speaks to the developer in the first person as an assistant. Rewrite these passages in the project's neutral voice, keeping every fact they carry.

Agent tooling the developer chose to keep is the project's content, not a trace: `CLAUDE.md`, `AGENTS.md`, skill folders, agent config, and code that integrates an AI API stay as they are.

## Steps

1. **Range.** Take the base commit from the arguments; otherwise use `git merge-base HEAD <default branch>`. Record the current `HEAD` SHA as the recovery point. Stash any uncommitted work with `git stash -u` so it stays out of the squash. Done when the base, the recovery SHA, and the commit list `<base>..HEAD` are known.
2. **Pushed check.** If any commit in the range is already on a remote, the squash needs a force-push. Show the developer which branch and commits are affected and wait for their go-ahead before continuing.
3. **Scan.** Read `git log <base>..HEAD` in full (bodies and trailers included) and every file the range touches, using `git diff --name-only <base>..HEAD`. Done when every trace in those messages and files is listed with its location.
4. **Clean.** Edit each listed trace out of the working tree. Change only the trace: behaviour, formatting, and surrounding text stay untouched.
5. **Squash.** Run `git reset --soft <base>`, stage everything, and commit once as the developer's own git identity. Write the message from the combined diff: a concise summary of what the change does, in the repo's existing commit style, with no trailers and no mention of cleanup, squashing, or agents.
6. **Verify.** Grep `git log --format='%an %ae%n%cn %ce%n%B' <base>..HEAD` and `git diff <base>..HEAD` for every trace class. Done when both come back clean; otherwise clean the remaining traces, `git commit --amend --no-edit` (or fix the message), and grep again.
7. **Restore.** `git stash pop` if step 1 stashed anything. Report the new commit SHA and the recovery SHA. Push only when the developer asks, using `--force-with-lease`.
