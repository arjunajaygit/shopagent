# Infrastructure CI

The workflow in `.github/workflows/infrastructure.yml` runs on pull requests to
`main`, pushes to `main` and `feature/W1-22-ci`, and manual runs once the workflow
is available on the default branch. It uses GitHub-hosted Ubuntu 24.04 runners.

| Check | Coverage |
| --- | --- |
| Infrastructure checks | Setup-helper safety tests, Compose configuration and shell script syntax |
| PostgreSQL integration | Empty-volume startup, health, host/container connections, rejected passwords, role permissions and data persistence after restart |

The database test creates a separate Compose project with random credentials and
removes its containers, network and volume afterward. No repository secrets or
developer `.env` files are needed. `.env.example` is used only to validate Compose
syntax; its placeholder passwords are not used to start a database.

The workflow has read-only repository permissions. Checkout is pinned to a commit
and does not persist Git credentials. Time limits bound each job, and a newer run
on the same branch or PR cancels the older run. A forcibly interrupted database
test may not finish cleanup; its GitHub-hosted runner is discarded after the job.

## Reproduce a failure locally

From the repository root, with Python 3, Docker Compose and the `psql` client:

```sh
python3 -B -m unittest discover -s infra/scripts -p test_setup_local.py -v
docker compose --env-file .env.example config --quiet
sh -n infra/scripts/check-db.sh
python3 -u infra/scripts/check-fresh-start.py
```

Open the failed check on the pull request and expand the failing step to see its
output. Fix the cause and push the correction to the same branch. GitHub runs the
checks again. Fork PRs may need a maintainer to approve the workflow run.

## Pending work

Local verification on 27 October 2026 passed all five setup-helper tests, Compose
validation, shell syntax and the isolated PostgreSQL test. The workflow YAML also
parsed successfully. A full workflow-linter check and a GitHub-hosted run remain
unverified.

- Verify the first GitHub Actions run; local tests alone do not verify the runner.
- Add backend tests/lint and frontend build/lint after their files and commands arrive.
- Confirm required-check names with Sonali before she configures merge rules.
- Add dependency and container scanning separately; these checks do not scan vulnerabilities.

W1-22 remains in progress until the agreed application checks are also implemented
and verified on a pull request.
