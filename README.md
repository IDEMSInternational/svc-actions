# svc-actions

The CI that every svc book on git.idems.international shares. A book's own
workflow files only call these:

| Workflow | Called from a book's | Does |
|---|---|---|
| [`build.yml`](.forgejo/workflows/build.yml) | `site-archive.yml`, on push | builds a base, or resolves and builds a variant; packages the site; publishes it from `main` |
| [`site-edit.yml`](.forgejo/workflows/site-edit.yml) | `site-edit.yml`, on `issues: opened` | turns an `[edit]` issue from the site into a pull request |
| [`site-feedback.yml`](.forgejo/workflows/site-feedback.yml) | `site-feedback.yml`, on `issues: opened` | labels a `[feedback]` issue, or folds it into the reader's earlier one |
| [`translation-site.yml`](.forgejo/workflows/translation-site.yml) | a translation repository's workflow, after its translate job | builds each translated book with the tool, comments and edits going to the translation repository or the source book; packages; publishes from `main` |
| [`site-translation-edit.yml`](.forgejo/workflows/site-translation-edit.yml) | a translation repository's `site-translation-edit.yml`, on `issues: opened` | records a `[translation-edit]` issue's edits on a branch and opens a pull request |

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

## Translation repositories

A translation repository (IDEMSInternational/pretext-translate, e.g.
IDEMSInternational/minimal-svc-translate) translates books listed in its
`translate.config.json` and publishes them under its own pages path,
`<name>-<lang>/` per source (IDEMSInternational/svc#361). Its translate job
uploads the translated projects as the artifact `translated`; then:

```yaml
  site:
    needs: translate
    uses: IDEMSInternational/svc-actions/.forgejo/workflows/translation-site.yml@main
    secrets: inherit
```

On each page, comments and edits about the content go to the source book's
repository, and comments and edits about the translation go to the translation
repository. So the translation repository calls `site-feedback.yml` and
`site-translation-edit.yml` from its own workflows on `issues: opened`, and
each source repository needs the book's own `site-feedback.yml` and
`site-edit.yml` to handle what arrives there.

`translation-site.yml` inputs: `feedback` (default `true`), `artifact`
(`translated`), `target` (`html`), `svc-ref` (`main`). It needs
`TRANSLATE_READ_TOKEN` (or `VARIANT_BUILD_READ_TOKEN`), able to read svc and
every source, and `PAGES_TOKEN` to publish.

Edits to a translation are only recorded for now, in
`feedback-edits/translations/` on a branch: a temporary holding area until the
translators settle how they should be kept (IDEMSInternational/svc#358).

## Versions

Books call `@main`, so a change here or in svc's `main` reaches every book on
its next run. A book that needs to hold still can pin both: a tag or commit of
this repository in its `uses:` line, and an svc commit in `svc-ref`.

## Where it lives

Changes are reviewed on GitHub. Books call the mirror at
`git.idems.international/IDEMSInternational/svc-actions`, which pulls from
GitHub every 10 minutes.
