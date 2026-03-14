# Maintaining a custom fork

This guide documents the branch layout used in this fork so you can keep your
own customizations while still pulling in the latest OpenAI changes later.

## Branch layout

- `origin` is your fork on GitHub.
- `upstream` is the official `openai/codex` repository.
- `vendor/openai-main` is a read-only mirror of `upstream/main`.
- `main` is your long-lived custom branch.

Rule of thumb:

- never add custom commits to `vendor/openai-main`
- do all product work on `main` or on short-lived feature branches created from
  `main`

## One-time setup

Run these commands once after creating your fork clone:

```bash
git remote add upstream git@github.com:openai/codex.git
git fetch upstream
git branch -f vendor/openai-main upstream/main
git push -u origin vendor/openai-main
git checkout main
```

After this setup:

- `vendor/openai-main` tracks the official code line
- `main` stays available for your branded or custom version

## Start customizing

Keep `main` as the branch that represents your product.

If you want a short-lived working branch for a feature, create it from `main`:

```bash
git checkout main
git checkout -b feature/my-custom-change
```

When the feature is ready, merge or rebase it back into `main` using your normal
workflow.

## Upgrade from upstream later

When OpenAI updates `main`, refresh the vendor branch first and only then move
your custom branch on top of it:

```bash
git fetch upstream

git checkout vendor/openai-main
git reset --hard upstream/main
git push --force-with-lease origin vendor/openai-main

git checkout main
git rebase vendor/openai-main
git push --force-with-lease origin main
```

What this does:

- `vendor/openai-main` becomes an exact copy of the latest upstream code
- your custom commits on `main` are replayed on top of the new upstream base

## Safety rules

- Never commit custom work directly to `vendor/openai-main`.
- Only use `git reset --hard` on `vendor/openai-main`, never on `main`.
- Keep custom commits small and focused so rebases stay manageable.
- Prefer runtime-loaded customization when possible:
  - config
  - prompts
  - skills
  - hooks
  - plugins
- If you must change source code, separate refactors from behavior changes.

## Quick checks

These commands are useful to verify the setup:

```bash
git remote -v
git branch --list
git rev-parse main
git rev-parse vendor/openai-main
git status --short
```

Right after initialization, `main` and `vendor/openai-main` should point to the
same commit. Over time they will diverge as you add your own custom commits to
`main`.
