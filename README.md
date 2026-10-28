# svc-actions

The CI that every svc book on git.idems.international shares. A book's own
workflow files only call these:

| Workflow | Called from a book's | Does |
|---|---|---|
| [`build.yml`](.forgejo/workflows/build.yml) | `site-archive.yml`, on push | builds a base, or resolves and builds a variant; packages the site; publishes it from `main` |
| [`site-edit.yml`](.forgejo/workflows/site-edit.yml) | `site-edit.yml`, on `issues: opened` | turns an `[edit]` issue from the site into a pull request |
| [`site-feedback.yml`](.forgejo/workflows/site-feedback.yml) | `site-feedback.yml`, on `issues: opened` | labels a `[feedback]` issue, or folds it into the reader's earlier one |

This repository holds workflow YAML only. The code it runs lives in
[svc](https://github.com/IDEMSInternational/svc): the svc-id HTML layer, its
build wrapper, the site packager, the feedback tool and `site_feedback.py`.
svc is private, so the workflows fetch it at build time with the organisation
secret `VARIANT_BUILD_READ_TOKEN`. This repository is public because Forgejo
only lets a workflow call another repository's workflow when that repository is
public. The decision is IDEMSInternational/svc#355.

## Calling it

```yaml
jobs:
  site:
    uses: IDEMSInternational/svc-actions/.forgejo/workflows/build.yml@main
    secrets: inherit
    with:
      feedback: false
```

The master copies of a book's call files are in svc, under
`scripts/base/.forgejo/workflows/`. `make_base.py` and `make_variant.py`
write them into new books.

`build.yml` inputs:

| Input | Default | |
|---|---|---|
| `feedback` | `false` | add the comment and edit tool to the site; keep `site-edit.yml` and `site-feedback.yml` in the book |
| `max-ptx-errors` | (strict) | tolerate a non-zero exit from pretext if it wrote the site and logged at most this many `PTX:ERROR` lines |
| `svc-ref` | `main` | the svc branch, tag or commit to build with |

`site-edit.yml` and `site-feedback.yml` take `svc-ref` too.

## Versions

Books call `@main`, so a change here or in svc's `main` reaches every book on
its next run. A book that needs to hold still can pin both: a tag or commit of
this repository in its `uses:` line, and an svc commit in `svc-ref`.

## Where it lives

Changes are reviewed on GitHub. Books call the mirror at
`git.idems.international/IDEMSInternational/svc-actions`, which pulls from
GitHub every 10 minutes.
