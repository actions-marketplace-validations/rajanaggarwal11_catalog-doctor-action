# catalog-doctor-action

Run [catalog-doctor](https://github.com/rajanaggarwal11/catalog-doctor) in CI. Catches the dependency drift a pnpm monorepo accumulates quietly — half-adopted catalogs, two versions of the same library, internal packages that can resolve to the registry, imports nothing declares — and writes a summary table into the workflow run.

```yaml
- uses: rajanaggarwal11/catalog-doctor-action@v1
```

That's the whole setup. No pnpm install, no Node setup, no config.

## What you get

The human-readable report lands in the job log, and a table lands in the run's **Summary** tab, so a failure is legible without opening logs:

> ## catalog-doctor
>
> **3 findings** — 2 error(s), 1 warning(s), 2 fixable.
>
> | | Rule | Dependency | Problem |
> | --- | --- | --- | --- |
> | 🔴 | `catalog-drift` | `zod` | zod resolves through the catalog in 1 place and directly in 1 |
> | 🔴 | `version-mismatch` | `lodash` | lodash is declared at 2 different versions |
> | 🟡 | `catalog-candidate` | `date-fns` | date-fns is declared as ^3.6.0 in 3 packages but is not in the catalog |

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `version` | `latest` | Which catalog-doctor to run. Pin it if you want reproducible CI. |
| `working-directory` | `.` | Where to start. The workspace root is found by walking up. |
| `strict` | `false` | Fail on warnings as well as errors. |
| `only` | — | Run only these rules, comma-separated. |
| `ignore` | — | Skip these rules, comma-separated. |
| `exclude` | — | Globs to keep out of source scanning, e.g. `test/fixtures/**`. |
| `fix` | `false` | Apply every fix that needs no human decision, then report what is left. |
| `fail-on-findings` | `true` | Set `false` to report without blocking the build. |

## Outputs

`total`, `errors`, `warnings`, `fixable`, and `report` — the path to the JSON report on the runner, if you want to upload or post-process it.

## Recipes

**Block the build only on real breakage, report the rest.**

```yaml
- uses: rajanaggarwal11/catalog-doctor-action@v1
  with:
    exclude: "test/fixtures/**"
```

Errors fail; warnings show up in the summary and let the build through. That is the default, and it is the right setting for adopting this on an existing monorepo.

**Tighten once you're clean.**

```yaml
- uses: rajanaggarwal11/catalog-doctor-action@v1
  with:
    strict: true
```

**Report without ever blocking**, while you work through a backlog:

```yaml
- uses: rajanaggarwal11/catalog-doctor-action@v1
  with:
    fail-on-findings: false
```

**Open a pull request with the automatic fixes.**

```yaml
- uses: actions/checkout@v5
- uses: rajanaggarwal11/catalog-doctor-action@v1
  with:
    fix: true
    fail-on-findings: false
- uses: peter-evans/create-pull-request@v7
  with:
    title: "chore: tidy the pnpm catalog"
    branch: chore/catalog-doctor
```

`--fix` only touches what has exactly one correct answer. It leaves version mismatches alone, because which range wins is a judgement about breaking changes and not a lookup — so the PR is always reviewable.

**Use the outputs.**

```yaml
- uses: rajanaggarwal11/catalog-doctor-action@v1
  id: doctor
  with:
    fail-on-findings: false
- run: echo "${{ steps.doctor.outputs.fixable }} of ${{ steps.doctor.outputs.total }} are fixable"
```

## Notes

It runs `npx catalog-doctor`, so it needs Node on the runner — every `ubuntu-*`, `macos-*` and `windows-*` image has it. It does not install your dependencies, because it reads manifests rather than `node_modules`; put it before or after your install step, whichever you prefer.

## License

[MIT](./LICENSE) © Rajan Aggarwal
