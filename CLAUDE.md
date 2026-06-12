# CLAUDE.md — filmdrop-ui (LGND fork)

## PUBLIC REPOSITORY — read before every commit

This is a **public** GitHub fork of
[Element84/filmdrop-ui](https://github.com/Element84/filmdrop-ui). Everything
pushed here is publicly visible: code, comments, docs, tests, commit
messages, and branch names.

Never commit or reference:

- Secrets, tokens, or credentials of any kind
- AWS account IDs, bucket names, or internal endpoints (API Gateway IDs,
  CloudFront domains, internal hostnames)
- Links to private LGND repositories, or their issue/PR numbers
- Internal planning details or content from private documents

Example/test values must be synthetic (`tiler.test`, `stac.test`,
`dynamodb://region/table:id`). Review the staged diff **and the commit
message** against this list before committing. When in doubt, leave it out —
context that needs privacy belongs in the private deployment/planning repos,
not here.

## Branch model

- `main` — clean mirror of upstream; never commit LGND work here. Sync:
  fast-forward from upstream, push.
- `main-lgnd` — the LGND line (what deploys). Rebase onto `main` after
  upstream syncs; force-push with `--force-with-lease`.
- Fixes useful beyond LGND should be PR'd to upstream Element84/filmdrop-ui,
  not accumulated on `main-lgnd`.

LGND-specific changes are documented in `LGND-README.md` (public-safe
wording only).

## Toolchain

Node (version in `.nvmrc`), vitest, eslint, prettier. Husky pre-commit runs
the prettier check and remark on markdown (max 200-character lines) — run
`npx prettier --write` on what you touch and keep markdown lines short.
