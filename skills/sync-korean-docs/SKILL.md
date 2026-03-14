---
name: sync-korean-docs
description: Sync `origin/main` into the `korean` branch and update Korean docs under `docs/ko-KR` to match changed English docs. Use when asked to sync the forked main branch, merge main into korean, translate new or updated docs, update ko-KR sync state, or push Korean doc sync work.
---

# Sync Korean Docs

Use this skill when the task is:

- sync the forked repository before translation work
- merge `main` changes into `korean`
- translate new or updated docs into `docs/ko-KR`
- update Korean docs links so translated docs point to translated pages
- update `docs/.i18n/sync-state.ko-KR.json`

## Required workflow

Do this in order.

1. Confirm the current branch and sync baseline.
2. Sync the forked `main` branch from GitHub first (`origin/main`).
3. Update local `main` to match `origin/main`.
4. Switch to `korean` and merge local `main` into it.
5. Resolve merge conflicts first, then translate docs.
6. Update `docs/.i18n/sync-state.ko-KR.json` only after translation is complete.
7. Commit only the translation/sync files.
8. Push `korean`.

Do not skip the merge step. The normal flow is:

```bash
git fetch origin main
git checkout main
git pull --ff-only origin main
git checkout korean
git merge main
```

Run the fetch/pull on `main`, then merge from `korean`.

## Read before editing

Always read:

- `docs/.i18n/sync-state.ko-KR.json`
- the local `main` vs `korean` docs diff after merge
- changed English docs under `docs/`
- matching Korean docs under `docs/ko-KR/`
- the English diff since the last sync SHA

Useful commands:

```bash
cat docs/.i18n/sync-state.ko-KR.json
git diff <LAST_SYNC_SHA>..origin/main --name-only -- docs/
git diff <LAST_SYNC_SHA>..origin/main -- docs/<path>.md
git diff main -- docs/
```

Ignore:

- `docs/zh-CN/**`
- non-doc changes unless the user explicitly asked for more

## Translation rules

- Keep code blocks, CLI commands, JSON keys, YAML keys, env vars, model IDs, config keys, and tool names in English.
- Translate frontmatter fields such as `title`, `summary`, `sidebarTitle`, and `read_when`.
- Preserve the existing tone and terminology used in nearby `docs/ko-KR` files.
- Update existing Korean pages incrementally when only parts changed.
- Create a new Korean file when the English page has no `docs/ko-KR/...` counterpart.

## Korean link policy

When editing Korean docs:

- internal Mintlify links should normally be `/ko-KR/...`
- if an absolute docs URL is used inside a Korean page, rewrite it to:
  `https://docs.openclaw.kr/ko-KR/...`

Examples:

- `https://docs.openclaw.ai/install` -> `https://docs.openclaw.kr/ko-KR/install`
- `/tools/browser` -> `/ko-KR/tools/browser`

## New file mapping

Map English docs to Korean docs like this:

- `docs/foo/bar.md` -> `docs/ko-KR/foo/bar.md`

Do not create fake locale nesting such as `docs/ko-KR/ja-JP/...`.

## Post-translation cleanup

Before finishing, scan the edited Korean docs for accidental full-file markdown wrappers.

Bad pattern:

- file starts with ```markdown` or ` ``markdown`
- file ends with a matching closing fence that wraps the whole document

This sometimes happens when translated markdown gets pasted back as a fenced block.
If found, remove only the outer wrapper and keep legitimate inner code fences.

Useful check:

`````bash
python3 - <<'PY'
from pathlib import Path
for p in sorted(Path('docs/ko-KR').rglob('*.md')):
    lines = p.read_text().splitlines()
    if lines and lines[0] in ('```markdown', '````markdown'):
        print(p)
PY
`````

Also compare `docs/ko-KR` against current English doc paths and remove stale Korean pages
whose English source path no longer exists because the doc moved.

Typical examples:

- old root page moved under `help/`, `tools/`, `providers/`, `automation/`, or `reference/`
- legacy path still exists only in `docs/ko-KR/...`
- `docs/docs.json` or Korean links still reference the old path

Cleanup rules:

- if the English source path is gone and the new Korean destination already exists, delete the stale old Korean file
- remove stale `docs/docs.json` Korean nav entries for removed pages
- update Korean links to the new path

Useful check:

```bash
python3 - <<'PY'
from pathlib import Path
root = Path('docs')
ko = root / 'ko-KR'
for p in sorted(ko.rglob('*.md')):
    rel = p.relative_to(ko)
    if not (root / rel).exists():
        print(rel)
PY
```

## Finishing rules

When all translations are complete:

1. set `last_sync_sha` to `git rev-parse origin/main`
2. set `last_sync_date` to the current ISO-8601 timestamp
3. commit with `scripts/committer`
4. push `origin korean`

Recommended commit message format:

```bash
scripts/committer "Docs: sync Korean docs from main (<SHORT_SHA>)" <files...>
```

## Safety rules

- Work file-by-file. Read English, read diff, read Korean, then edit.
- For large syncs, batch by small diffs first, then medium/large rewrites.
- Do not overwrite existing Korean content blindly.
- Do not mark sync complete before the changed docs are actually translated.
- If merge conflicts touch Korean docs, resolve the merge first, then re-check the final English side before translating.
