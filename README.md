# actions

Shared GitHub Actions for eugenius1's repos.

## Claude review

`claude-review/code` runs the code-review plugin's `/code-review` and
`claude-review/security` runs `/security-review` on a pull request, and each
posts its findings as inline comments. A repo calls them from its own
`.github/workflows/claude-review.yml`, one job each:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  id-token: write

jobs:
  review:
    name: Code review
    if: >-
      !github.event.pull_request.draft &&
      github.event.pull_request.head.repo.full_name == github.repository &&
      github.actor != 'dependabot[bot]'
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: eugenius1/actions/claude-review/code@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

  security:
    name: Security review
    if: >-
      !github.event.pull_request.draft &&
      github.event.pull_request.head.repo.full_name == github.repository &&
      github.actor != 'dependabot[bot]'
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: eugenius1/actions/claude-review/security@main
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

The repo needs the `CLAUDE_CODE_OAUTH_TOKEN` secret (made with
`claude setup-token`).

These are composite actions rather than a reusable workflow because
`claude-code-action` checks the calling workflow file against the repo's
default branch before it runs. Keeping that file in each repo keeps the check
as it was.

Repos use `@main`, so a change merged here reaches every repo's next PR at
once, and no PR review by Claude checks it first.
