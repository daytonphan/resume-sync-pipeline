# Resume Sync Pipeline

Automatically compile a LaTeX resume, generate a redacted public variant,
and keep it in sync across a portfolio site, personal storage, and more. This pipeline relies on a
security model where a compromise of your source repo can never propagate
to your production site.

## What this does

1. Edit your resume `.tex` file (this template assumes an Overleaf → GitHub
   sync as the trigger, but any `git push` works).
2. A GitHub Actions workflow compiles the full PDF, then generates and
   compiles a second **redacted** PDF with sensitive content (e.g. a phone
   number) stripped out via a configurable pattern.
3. Downstream consumers, like your portfolio site, personal cloud storage, etc.
    independently **pull** the redacted or full PDF on a schedule, using a
   read-only, single-repo-scoped credential. The source repo never holds a
   credential that can write anywhere but itself.

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the full reasoning
behind the pull-based design and other hardening choices. Although this design requires more setup, it is essential for security.

## Quickstart

### 1. Source repo (compiles your resume)

- Copy [`.github/workflows/compile-resume.yml`](.github/workflows/compile-resume.yml)
  into your resume's repo.
- Edit the `MAIN_TEX_FILE` and `REDACT_PATTERN` values at the top of the
  file (see [Customizing redaction](#customizing-redaction) below).
- In that repo: **Settings → Actions → General → Workflow permissions** →
  select **"Read and write permissions"**.
- Push a `.tex` change (or run the workflow manually) to test.

### 2. Downstream repo (e.g. your portfolio site)

- Generate a fine-grained token (github.com/settings/tokens?type=beta)
  scoped to **only** your source repo, with **Contents: Read-only**
  permission.
- Add it as a secret named `RESUME_READ_PAT` in your downstream repo.
- Copy [`.github/workflows/pull-resume.yml`](.github/workflows/pull-resume.yml)
  into that repo, and fill in `SOURCE_REPO`, `SOURCE_FILE_PATH`, and
  `DEST_PATH` at the top.
- If your site has its own build/deploy workflow, make sure it either
  triggers on changes to `DEST_PATH`, or runs on a schedule shortly after
  this one.

### 3. (Optional) Personal cloud backup

See [`shortcuts/README.md`](shortcuts/README.md) for a fully on-device,
pull-based setup using iOS Shortcuts, which requires no cloud credential.

## Customizing redaction

The default `REDACT_PATTERN` assumes your resume uses a FontAwesome icon
command (`\faPhone`) on the same line as your phone number, and deletes any
line matching that pattern. If your resume is structured differently:

- To strip a whole line, set `REDACT_PATTERN` to any regex unique to that
  line.
- To redact a value in-place instead of deleting the whole line (e.g. keep
  an icon but blank out just the number), replace the `sed '/pattern/d'`
  command in the workflow with a substitution, e.g.
  `sed 's/([0-9]{3}) [0-9]{3}-[0-9]{4}//'`.
- Test locally first: `sed '/YOUR_PATTERN/d' main.tex` and check the output
  before relying on it in CI.

## Known limitations

- The redaction step is pattern-based, not semantic. So if you restructure
  your resume's header later, double check the pattern still matches
  correctly.
- Pull-based sync means updates take up to your polling interval to appear
  downstream, and do not occur instantly. See `docs/ARCHITECTURE.md` for why this is a
  deliberate tradeoff.

## License

MIT — see [`LICENSE`](LICENSE).
