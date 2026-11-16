---
name: how-to-commit
description: Use when writing or reviewing commit messages, preparing a commit,
  or making an authorized commit. Guides atomic commits, Conventional Commits
  messages, and verification of the commit.
---

# How to Commit

Use these rules to make commits that a reviewer can read, test, and revert
one at a time.

## Make Atomic Commits

An atomic commit contains one coherent change. A reviewer can review it, test
it, and revert it as a unit.

- Keep the implementation, its tests, its docs, and the matching manifests and
  lockfiles in one commit.
- Split changes that are independent. Do not split one change by file type.
- Do not defer required tests or prerequisites to a later commit.

A fix for expired password-reset links belongs with its regression test and any
required docs. An unrelated login-page redesign belongs in another commit.

Before you choose the contents, inspect the request, the code, and the staged,
unstaged, and untracked changes. Preserve unrelated work.

## Write the Message

Use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
unless the repository has established a different pattern. Written guidance,
tool configuration such as a commitlint config, or consistent recent history can
establish a pattern. Within Conventional Commits, match the types, the scopes,
and the wording of similar earlier commits.

```text
type(scope)!: description
```

The type and the description are required. Choose the type from the effect of
the change:

| Type       | Use                                                                 |
| ---------- | ------------------------------------------------------------------- |
| `feat`     | A new feature.                                                      |
| `fix`      | A bug fix, with its regression tests.                               |
| `refactor` | A restructure that does not change behavior.                        |
| `perf`     | A change that improves performance.                                 |
| `docs`     | A change to documentation only.                                     |
| `test`     | A change to tests only, such as coverage for existing behavior.     |
| `build`    | A change to the build system or to dependencies.                    |
| `ci`       | A change to the CI configuration or its scripts.                    |
| `chore`    | Maintenance that fits no other type, such as a `.gitignore` update. |
| `revert`   | A commit that undoes an earlier one. Identify it and the reason.    |

Types other than `feat` and `fix` are conventions. Check which types the
repository allows, and whether a type affects releases.

- **Scope:** A scope names the part of the product that the change affects.
  Choose the name for the reader of the changelog. When the repository has
  no changelog, the readers are its developers.
  - When the changelog is for end users, use a name that they know, such as a
    feature, a command, or a page.
  - When the changelog is for developers, use the name of the package or the
    module.
  - Use the scope that earlier commits use for the same part. Do not add a
    second name for it.
  - Omit the scope when the change affects several parts, or when no name helps
    the reader.
  - Do not use a type, such as `test` or `docs`, as a scope.
- **`!`:** Add it only for a breaking change.
- **Description:** Write a short imperative description with no final period.
  Keep the casing of identifiers. Describe the resulting behavior or the
  technical purpose, not the files or the work history.

**Before:**

```text
fix: update writer.go and add tests
```

**After, for end users:**

```text
fix(export): write dates in UTC
```

**After, for developers:**

```text
fix(csvwriter): write dates in UTC
```

The first description lists files. The other two state the behavior that
changed. An end user knows the `export` command. A developer knows the
`csvwriter` package that implements it.

Add a body only when it gives reasons, tradeoffs, or migration steps. Separate
the first line, the body, and the footers with blank lines. Do not add a
`Co-Authored-By` trailer.

Any type can be a breaking change. Mark it with `!`, with a `BREAKING CHANGE:`
footer, or with both, as the repository requires. With `!` alone, state the
incompatibility in the description. When a squash merge replaces the message,
keep the migration information in the final message.

For example, the removal of a command-line flag breaks scripts that use it.
The body gives the reason, and the footer tells users how to migrate:

```text
feat(export)!: remove the --out flag

The --output flag replaced --out in v1.4. Since then, --out has printed a
deprecation warning.

BREAKING CHANGE: export no longer accepts --out. Use --output with the
same value.
```

## Verify and Commit

Do only what the user authorized:

- A request for a message, or for a proposal to split changes, does not
  authorize a commit.
- A request to commit does not authorize a rewrite of history, a push, or a
  merge.
- Follow the user's choice between an amendment and a new commit.

Then follow these steps:

1. **Stage:** Stage explicit paths or reviewed hunks. Preserve unrelated staged
   work. If you cannot tell which change a staged file belongs to, ask before
   you unstage it.
2. **Inspect:** Read the staged diff. Check that it is correct, and that the
   message describes it.
3. **Test:** Run the checks for the staged contents. If unstaged changes could
   affect the checks, test the staged contents in isolation, and do not disturb
   the unstaged work. Reuse earlier results that are still valid.
4. **Validate the message:** Run the repository's message tooling, such as
   commitlint, when it is installed. Fix a rejected message. Do not bypass hooks
   or weaken rules.
5. **Commit:** Create the commit. Then check that the commit holds only
   the staged changes, and that the unrelated work is intact.

Report these items:

- The commit ID and its first line, or the proposed message when you did not
  commit.
- The checks that you ran, and the checks that were unavailable.
- Any unresolved findings.
