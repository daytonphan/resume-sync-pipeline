# Architecture

## The problem

It is very tedious to keep a resume up to date across multiple destinations. One may want resumes on their portfolio site, a personal backup, a recruiter-facing copy, etc. However, it is very easy to let some resumes in some destinations become stale. 
Each destination often needs slightly different content too (a public site should omit some personal info), so simple file-copying becomes insufficient.

## Trust model (pull instead of push)

The core design decision in this pipeline is that **less-trusted components should
never hold credentials that can write to more-trusted ones.**

A naive version of this pipeline would have the source resume repo push
directly to your portfolio repo and to cloud storage. That would simpler, but it
means the source repo (which is the most
likely place for something to go wrong) holds a credential capable of
modifying your live site, or allowing an attacker to push a malicious file to your cloud storage.

This pipeline inverts that:

```
┌─────────────────┐         ┌──────────────────┐         ┌──────────────┐
│  Resume source   │  push   │  Compiles + only  │  PULL   │  Portfolio /  │
│  repo (private)  │ ──────► │  writes to itself  │ ◄────── │  cloud, etc.  │
└─────────────────┘         └──────────────────┘         └──────────────┘
                                                     (read-only, scoped token)
```

- The **source repo's** workflow uses only the default `GITHUB_TOKEN`,
  automatically scoped to itself. It has no credential that can reach
  anywhere else.
- Each **downstream consumer** (portfolio, cloud backup, etc.) independently
  *pulls* from the source repo on a schedule, using a fine-grained,
  read-only, single-repository token.

**Consequence:** if the source repo is ever compromised by a malicious
dependency, a leaked token, or a supply-chain issue in a third-party action,
the damage is contained. There is nothing in that repo capable of
writing to your production site or your personal cloud storage.

**Tradeoff:** there is no near-instant propagation. A push-based model can
update a downstream site quickly. However, polling on a schedule means
waiting up to your poll interval. This template defaults to once daily,
timed to run shortly before whatever site-rebuild schedule one might have,
so it stays cheap on CI minutes. You may shorten the cron interval if you want
faster updates, but this trades
security isolation for latency, not getting both for free.

## Other hardening choices

- **No third-party GitHub Actions.** LaTeX is compiled by installing
  TeXLive directly via `apt-get` and calling `latexmk`, rather than using a
  third-party wrapper action. Fewer parties to trust, at the cost of a
  slower cold install per run versus a pre-built container image.
- **Least-privilege tokens.** Every credential in this pipeline is scoped to
  exactly one repository and exactly the permission it needs (e.g.
  "Contents: Read-only" for pulling), using GitHub's fine-grained personal
  access tokens rather than broad classic tokens.
- **On-device credentials for personal sync.** The optional iCloud/local
  backup step (see `shortcuts/README.md`) uses an on-device automation
  rather than a cloud CI secret with real account credentials. Thus, no
  third-party service ever holds a token capable of touching your personal
  cloud storage.
