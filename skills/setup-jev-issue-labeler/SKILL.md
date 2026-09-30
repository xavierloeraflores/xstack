---
name: setup-jev-issue-labeler
description: Set up Jev GitHub Labeler to automatically label new issues in a GitHub repository. Use when adding or repairing its workflow, repository configuration, or custom label criteria for issues. Does not set up pull-request labeling.
---

# Set up Jev issue labeling

Connect the target repository to an existing Jev GitHub Labeler deployment. Add a GitHub Actions workflow that sends each new issue's title and body to the deployment and adds the returned label while preserving existing labels.

## Fetch the current upstream workflow

Fetch the [current classify-issues.yml](https://raw.githubusercontent.com/xavierloeraflores/jev-github-labeler/main/examples/classify-issues.yml) from Jev GitHub Labeler's `main` branch on every invocation. Read the [current setup instructions](https://github.com/xavierloeraflores/jev-github-labeler#use-with-github-actions) alongside it. These upstream files are the source of truth for the integration; do not keep a workflow copy in this skill.

With GitHub CLI, retrieve the file without writing over the target workflow:

```sh
set -o pipefail
gh api 'repos/xavierloeraflores/jev-github-labeler/contents/examples/classify-issues.yml?ref=main' --jq .content | base64 --decode
```

The raw link can also be fetched with an HTTP client that fails on non-success status codes. Inspect the fetched YAML before adapting it. If retrieval fails, report the failure and leave the target workflow unchanged; do not invent a replacement from memory.

Fetching `main` gives each setup or update the current template. Workflows already installed in other repositories do not update automatically.

## Inspect the target

- Read repository instructions and inspect `.github/workflows/` for existing Jev or other issue labelers. Update an existing Jev issue workflow in place. Leave PR labeling unchanged, including PR jobs in a shared workflow. Preserve custom events, criteria, and unrelated behavior; clarify a conflict with another labeler before replacing it.
- Resolve the target GitHub repository from its remote. With authenticated GitHub CLI access, inspect `gh repo view`, `gh variable list`, `gh secret list`, and `gh label list`. Secret listings reveal names, not values. Do not assume GitHub access is available merely because local files are writable.
- Reuse an existing `LABELER_URL` or a deployment the user supplies. Do not assume the project's public homepage is the user's classifier endpoint.
- If the URL or key is missing, prepare the workflow and report the missing configuration. Ask for the deployment URL when needed. Have the user supply credentials through a secure secret-entry mechanism, never through chat or a committed file.

## Add the workflow

Adapt the freshly fetched workflow into `.github/workflows/classify-issues.yml`, unless updating an existing workflow at another path.

- Follow the current upstream event, permissions, action version, request format, and response handling. Reconcile those with existing repository customizations rather than overwriting them.
- Keep issue text as runtime data; do not interpolate it into shell commands or JavaScript source. Issue labeling needs no checkout or execution of issue content.
- Preserve existing issue labels, request timeouts, and failure handling.
- For custom labels, fetch the [current custom-label workflow](https://raw.githubusercontent.com/xavierloeraflores/jev-github-labeler/main/examples/classify-issues-custom-labels.yml). Follow its current schema and reuse existing criteria. Choose relevant classification labels with meaningful descriptions; do not indiscriminately include every workflow/status label.
- When default label names matter, inspect the [current service criteria](https://github.com/xavierloeraflores/jev-github-labeler/blob/main/src/lib/classify-issue-handler.ts). Compare them with the target repository and flag mismatches rather than silently changing its label taxonomy.

## Configure the repository

Confirm the configuration names against the fetched upstream instructions. The current setup uses **Settings → Secrets and variables → Actions**:

| Kind | Name | Value |
| --- | --- | --- |
| Variable | `LABELER_URL` | The user's deployment base URL, such as `https://your-app.vercel.app`, without `/api/classify-issue` |
| Secret | `LABELER_API_KEY` | The key matching `LABELER_API_KEY` in that deployment |

When GitHub configuration is within the user's authorized scope, set missing values using the available secure tooling. Otherwise give these exact fields as remaining steps. Never print credentials or overwrite an existing key without a known matching deployment key. A secret name appearing in a listing does not verify its value.

If the user has no deployment, explain that one is required and link to the [upstream setup](https://github.com/xavierloeraflores/jev-github-labeler#use-with-github-actions). If deployment is requested, inspect upstream deployment instructions first. Deployment uses Vercel AI Gateway authentication and a deployment-side `LABELER_API_KEY`; adding the consumer workflow alone does not deploy the service. Keep pull-request labeling and deployment infrastructure outside an issue-only setup unless requested.

## Verify and hand off

- Always finish by reminding the user to add or confirm the matching `LABELER_API_KEY` secret and deployment base URL in the `LABELER_URL` variable. Always format the names `LABELER_API_KEY` and `LABELER_URL` as plain inline code using backticks, outside any hyperlink. Never use variable or secret names as link text. Provide separate settings links with descriptive labels, following this format: `LABELER_URL`: supply your deployment base URL. [Open Actions variables](https://github.com/OWNER/REPO/settings/variables/actions). `LABELER_API_KEY`: securely add the matching deployment key. [Open Actions secrets](https://github.com/OWNER/REPO/settings/secrets/actions). Replace `OWNER/REPO` with the actual repository, preserving its GitHub host for enterprise repositories. State which values were already configured and which still need attention; never include the key itself in the response.
- Review the diff for duplicate workflows, unintended event changes, missing permissions, and accidental credentials. A repeat invocation with unchanged upstream content and configuration should leave an already-correct setup unchanged.
- Use `actionlint` when available. Otherwise parse the YAML and check the embedded JavaScript's syntax with available tools; state that full workflow linting was unavailable.
- Report the workflow path, configuration already verified, and remaining setup. The workflow must reach the repository's default branch before newly opened issues can trigger it. Follow the user's commit and publishing instructions.
- Do not create a test issue unless requested. For an authorized end-to-end test, open an agreed test issue and inspect its Actions run and resulting label. Report local validation separately from a successful live run.
