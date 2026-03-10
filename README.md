# testing-light-system

A demo repository for testing the [PR Traffic Light System](https://github.com/jpkapinha/pr-light-system) — a GitHub Actions workflow that automates PR management using traffic light labels.

---

## How It Works

Every Pull Request must have one of three labels before it can be merged:

| Label | Automated Action |
|---|---|
| `red-light` | Automatically requests a review from the repo owner. Merge is blocked until approved. |
| `green-light` | Enables auto-merge. PR merges automatically once all required checks pass. |
| `yellow-light` | No automation. Validation passes but a human must merge manually. |
| *(no label)* | ❌ PR is **blocked**. Workflow fails until a valid label is added. |

---

## Full Setup Guide

### Step 1 — Create the Labels

1. Go to **Issues > Labels > New label**
2. Create the following three labels:

| Label name | Suggested color | Purpose |
|---|---|---|
| `red-light` | `#EE0701` | Triggers review request |
| `green-light` | `#0075CA` | Triggers auto-merge |
| `yellow-light` | `#E4E669` | Manual merge required |

> ⚠️ Label names are case-sensitive. Use lowercase exactly as shown.

---

### Step 2 — Enable Auto-Merge

The `green-light` automation requires GitHub's native auto-merge feature to be turned on.

1. Go to **Settings > General**
2. Scroll to the **Pull Requests** section
3. ✅ Check **Allow auto-merge**

---

### Step 3 — Set Up Branch Protection on `main`

> ⚠️ **This step is required for auto-merge to work.** Without branch protection rules, GitHub rejects the auto-merge request entirely.

1. Go to **Settings > Branches > Add branch protection rule**
2. Set **Branch name pattern** to `main`
3. ✅ Enable **Require status checks to pass before merging**
4. Search for and add `Check Traffic Light Label` as a required status check
5. ✅ Enable **Require branches to be up to date before merging**
6. Click **Save changes**

> 💡 **Why is this required?** GitHub only allows auto-merge on PRs that have branch protection rules enabled. Without them, the `enablePullRequestAutoMerge` API call is rejected regardless of permissions. This is a GitHub platform constraint, not a workflow limitation.

---

### Step 4 — Add the Workflow File

1. Create `.github/workflows/traffic-light.yml` in your repository
2. Copy the contents from [.github/workflows/traffic-light.yml](./.github/workflows/traffic-light.yml)
3. Commit directly to `main`

The system is now live and triggers on every Pull Request event.

---

## Workflow File Notes

The workflow uses the **REST API** (`github.rest.pulls.update`) to enable auto-merge, with a GraphQL fallback. This is more reliable than GraphQL-only because the `GITHUB_TOKEN` provided by GitHub Actions supports the REST endpoint without additional configuration.

```yaml
permissions:
  pull-requests: write
  contents: write
```

These top-level permissions must be declared in the workflow for the token to have sufficient access.

---

## Troubleshooting

**`green-light` auto-merge fails immediately**
→ Make sure branch protection rules are enabled on `main` (see Step 3 above).
→ Make sure "Allow auto-merge" is checked in Settings > General.

**`red-light` review request fails**
→ GitHub does not allow the PR author to be assigned as their own reviewer.
→ Add a collaborator to the repo and update the `reviewers` list in the workflow.

**Workflow is not running at all**
→ Confirm the file is at exactly `.github/workflows/traffic-light.yml` (note the leading dot in `.github`).

**Label check fails even after adding a label**
→ Label names are case-sensitive: `red-light`, `green-light`, `yellow-light` — all lowercase, hyphenated.

**GitHub Actions not enabled**
→ Go to Settings > Actions > General and select "Allow all actions and reusable workflows".

---

## Customisation

### Change who receives the review request (red-light)

In `.github/workflows/traffic-light.yml`, find the `"Request Review (red-light)"` step and update:

```yaml
# One or more specific users:
reviewers: ['alice', 'bob']

# A GitHub team (use team slug):
team_reviewers: ['backend-team']
```

### Change the merge method (green-light)

Find `mergeMethod` in the workflow and replace the value:

| Value | Result |
|---|---|
| `MERGE` | Standard merge commit (default) |
| `SQUASH` | Squash all commits into one |
| `REBASE` | Rebase commits onto base branch |

---

## References

- [PR Light System source repo](https://github.com/jpkapinha/pr-light-system)
- [GitHub Actions — Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
- [GitHub — Managing auto-merge](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request)
- [GitHub — Branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
