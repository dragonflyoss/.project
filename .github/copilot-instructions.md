# Copilot Instructions for `dragonflyoss/.project`

This repository is the CNCF `.project` (dot-project) canonical metadata repository for [Dragonfly](https://d7y.io/) (`dragonflyoss`), managed alongside upstream [`cncf/automation`](https://github.com/cncf/automation/tree/main/utilities/dot-project) tooling.

## Architecture & Core Files

This is a declarative YAML/configuration repository with no application source code. All automation delegates to actions in `cncf/automation`:

- **`project.yaml`**: Canonical project metadata conforming to the CNCF `.project` schema (`schema_version: "1.0.0"`), including maturity history, primary repositories, package managers (Docker, Helm, Cargo), governance/security/legal document links, and CNCF Landscape categorization.
- **`maintainers.yaml`**: Roster of project maintainers under `project_id: "dragonfly"` and `org: "dragonflyoss"`, used by CNCF automation for drift detection and mailing-list synchronization.
- **`CODEOWNERS`**: GitHub code-owner review rules mirroring the maintainer roster.
- **`.github/workflows/validate.yaml`**: Runs `validate-project` and `validate-maintainers` on every pull request and pushes to `main`.
- **`.github/workflows/update-landscape.yml`**: Runs `cncf/automation/.github/actions/landscape-update` on pushes to `main` touching `project.yaml`, opening update PRs against `cncf/landscape` using `secrets.LANDSCAPE_REPO_TOKEN`.
- **`.github/workflows/check-links.yml`**: Biweekly scheduled workflow (and `workflow_dispatch`) that verifies all URLs and GitHub handles across the repository and manages tracking issues labeled `broken-links`.

## Validation Commands

There is no local build system in this repo. Validation is performed using the upstream Go validator from `cncf/automation`:

```bash
# Build or run the upstream validator (from github.com/cncf/automation/utilities/dot-project)
/path/to/validator -repo-root . -check-links
```

- Set `GITHUB_TOKEN` (or authenticate via `gh auth login`) before running `-check-links` to avoid GitHub API rate limits when resolving handles and repository URLs.
- Unconfirmable links are emitted as warnings rather than non-zero exit codes; inspect validator output directly even when exit status is `0`.

## Key Conventions & Invariants

1. **Keep `maintainers.yaml` and `CODEOWNERS` synchronized**: Whenever adding, removing, or renaming a maintainer handle in `maintainers.yaml` (`teams[name="project-maintainers"].members`), update the `* @handle ...` line in `CODEOWNERS` in the same change.
2. **Upstream Schema Compliance**: Changes to `project.yaml` and `maintainers.yaml` must conform to the [CNCF `.project` Schema Reference](https://github.com/cncf/automation/blob/main/utilities/dot-project/SCHEMA.md).
3. **GitHub Actions References & Permissions**:
   - Use readable release tags rather than commit hashes (e.g., `actions/checkout@v7.0.1`). Use `@main` for `cncf/automation` while it publishes no release tags; these branch references follow upstream changes.
   - `actions/checkout` steps must set `persist-credentials: false`. Use `fetch-depth: 0` in validation and landscape-update workflows so diff-aware checks can compare against base commits.
   - Keep workflow `permissions` minimal (`contents: read` by default; only `check-links.yml` grants `issues: write`).
4. **PR Validation Triggering**: Do not add `paths` filters to the `pull_request` trigger in `.github/workflows/validate.yaml`. `validate-project` and `validate-maintainers` must run on every PR (including documentation-only PRs) so required branch-protection status checks never stay stuck in a pending state.
5. **LFX Verification Setting**: `verify_maintainers` in `.github/workflows/validate.yaml` is intentionally set to `"false"` per upstream guidance while maintainers link their GitHub handles to LFIDs at `openprofile.dev`.
