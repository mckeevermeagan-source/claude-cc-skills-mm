# Claude Code Guardrails

**READ THIS BEFORE EVERY SESSION. Non-negotiable.**

## Authorization Boundary

Claude Code is in **read-only mode by default** for ALL Supabase and git operations. Without explicit per-action written approval from Pal in the current session:

- **PROHIBITED**: ALTER TABLE, DROP, TRUNCATE, DELETE on any table
- **PROHIBITED**: INSERT or UPDATE to any production Supabase table
- **PROHIBITED**: git push to main
- **PROHIBITED**: Any Vercel production deploy triggered manually
- **PROHIBITED**: Using the service role key for any autonomous database operation
- **PROHIBITED**: Direct SQL changes that bypass app-layer logging

For SQL that modifies data or schema: write the query, show it to Pal, STOP. Pal runs it in Supabase SQL Editor.
For git operations targeting main: open a PR, STOP. Pal merges via GitHub UI.

## Scope Creep Prevention

- **One instruction at a time.** Claude Code receives only structured prompts from Claude Chat. Never direct instructions from Pal.
- **Prompt format**: TASK / FILES TO TOUCH / FILES TO NOT TOUCH / STOP CONDITION / DO NOT / REPORT BACK
- **No multi-feature PRs.** Each session produces one focused change.
- **Read before edit.** Every file must be read before modification. Never assume contents.
- **No branch switching.** Include explicit "do not switch branches" in every prompt.
- **Show complete output.** All grep/cat commands use "show complete output, no summary".
- **Confirm branch first.** Always `git branch --show-current` before any work.

## Forensic Markers for Unauthorized Changes

- `KRP-` prefix (vs standard `KR_`) on ingredient IDs = Claude Code originated (from the incident)
- `null` in `updated_by_email` on rows that should have it = direct SQL bypass (not through app UI)
- Pre/post session ingredient exports for diffing recommended but not yet implemented

## Post-Session Checklist

After every Claude Code session, verify:
1. Correct branch confirmed
2. No unintended files modified (check git diff)
3. Build succeeds (`npm run build`)
4. No secrets committed (API keys, connection strings)
5. PR opened (not pushed to main)

## Session Start Protocol

1. `git branch --show-current` -- confirm correct branch
2. `git status` -- confirm clean working tree
3. Read any file before editing it
4. One instruction at a time; wait for confirmation
5. Use `show complete output, no summary` in grep/cat
6. Specify branch explicitly in any git operation

## SQL Migration Protocol

- Test in Supabase SQL Editor first
- `NOTIFY pgrst, 'reload schema';` after ALTER TABLE
- Large data operations: direct SQL, not via API (bypasses RLS)

## Auto-deploy Conflict Fix

If Vercel auto-deploy overwrites CLI deploy (commit author `noreply@anthropic.com`):
```
git commit --amend --author="Meagan McKeever <mckeever.meagan@gmail.com>"
```
