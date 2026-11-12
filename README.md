# Dragonfly `.project`

`.project` (dot-project) is a CNCF initiative to centralize and automate metadata management for all CNCF projects.
This repository holds the canonical metadata for [Dragonfly](https://d7y.io/) and is maintained by the Dragonfly maintainers with CNCF automation tooling.

## What's in this repo

| File                                     | Purpose                                                                                               |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `project.yaml`                           | Canonical project metadata (name, maturity, repositories, governance links, …)                        |
| `maintainers.yaml`                       | Maintainer and reviewer roster used for drift detection and mailing-list sync                         |
| `CODEOWNERS`                             | Requests reviews from project maintainers                                                             |
| `.github/workflows/validate.yaml`        | CI — validates `project.yaml` and `maintainers.yaml` on every PR                                      |
| `.github/workflows/update-landscape.yml` | Automatically proposes landscape updates when `project.yaml` changes                                  |
| `.github/workflows/check-links.yml`      | Every other week, checks all links and maintainer handles and keeps a `broken-links` issue up to date |

## Keeping metadata up to date

Open a pull request against this repository to update any metadata field.
The validate workflow checks schema correctness on every PR, including documentation-only changes, so required checks are not left pending by path filters.
On PRs and pushes, only newly added links and GitHub handles are checked; the scheduled link check covers all existing entries and can also be run manually.

To block merges, configure branch protection or a ruleset for `main` to require the `validate-project` and `validate-maintainers` checks and code-owner approval.
The workflows and [CODEOWNERS](CODEOWNERS) do not enforce these requirements by themselves.

When changing team membership, keep [maintainers.yaml](maintainers.yaml) and [CODEOWNERS](CODEOWNERS) in sync.
LFX verification is currently disabled in line with upstream guidance; maintainers should still connect their GitHub handle to their LFID at [openprofile.dev](https://openprofile.dev).

## Workflow configuration

Configure the `LANDSCAPE_REPO_TOKEN` repository secret with a token authorized to create update pull requests in `cncf/landscape`.
The landscape workflow uses this dedicated token rather than granting write access to this repository's `GITHUB_TOKEN`.
Validation needs only `contents: read`; the scheduled link check additionally needs `issues: write` to manage its report issue.

## Local validation

Build the upstream [validator](https://github.com/cncf/automation/tree/main/utilities/dot-project#validator), then run from this repository:

```bash
/path/to/validator -repo-root . -check-links
```

Use an authenticated GitHub token via `GITHUB_TOKEN` or `gh auth login` to avoid API rate limits.
Links that cannot be confirmed are reported as warnings, so a successful exit does not mean every link was verified.

## Resources

- [`.project` documentation](https://github.com/cncf/automation/tree/main/utilities/dot-project)
- [Schema reference](https://github.com/cncf/automation/blob/main/utilities/dot-project/SCHEMA.md)
- [CNCF Automation repository](https://github.com/cncf/automation)
