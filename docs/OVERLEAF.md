# Using this with Overleaf

The pipeline triggers on any `git push` of a `.tex` file, so Overleaf is
optional. If you do write your resume in Overleaf, this is how to connect it.

## Linking a project

Overleaf's GitHub sync requires a plan that includes it (a paid plan, or
access through a school or institution). The menu location changes between
Overleaf versions. Look for **Integrations → GitHub** in the project editor.

1. Open the project and go to the GitHub sync panel.
2. Choose **Create a GitHub repository**. Pick **Private** if your resume
   contains personal details, since the full (unredacted) PDF gets committed
   to this repo.
3. Add this template's `.github/workflows/compile-resume.yml` to the new
   repo (GitHub's web UI works: **Add file → Create new file**).

## Daily flow

1. Edit in Overleaf.
2. In the GitHub sync panel, **pull GitHub changes into Overleaf** first,
   then **push Overleaf changes to GitHub**.
3. The workflow compiles both PDFs and commits them back to the repo.

Overleaf never pushes on its own. Syncing is always just a button press.

## Known issue: "could not be automatically merged"

Because the workflow commits PDFs back to the default branch, the repo moves
forward without Overleaf knowing. If Overleaf then tries to push from an
older snapshot, it can't fast-forward, so it pushes your edit to a side
branch named like `overleaf-2026-07-27-2018` and shows:

> Your changes in Overleaf and GitHub could not be automatically merged.
> Please manually merge the overleaf-... branch into the default branch.

Nothing is lost. Merge the branch yourself:

```bash
git fetch origin
git checkout main
git pull
git merge origin/overleaf-2026-07-27-2018   # use the branch name from the error
```

If git reports a conflict, check which file it names:

- **`.github/workflows/...`**: Overleaf re-pushed a stale copy of a file it
  doesn't edit. Keep **your current** version and delete the conflict markers
  (`<<<<<<<`, `=======`, `>>>>>>>`) along with Overleaf's copy.
- **`main.tex`**: keep Overleaf's side, since that's your latest edit.

Then `git add`, `git commit`, `git push`, and use the continue button in
Overleaf's dialog.

**Avoiding it:** pull GitHub changes into Overleaf before every push, so
Overleaf has the bot's latest commit. If it still happens often, change the
workflow to commit PDFs to a separate branch (e.g. `build`) instead of the
default one. Overleaf only tracks the default branch, so it would never see
the bot's commits. Downstream consumers would then read from that branch.
