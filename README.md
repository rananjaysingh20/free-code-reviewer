# Free AI code reviewer (Jules + GitHub Actions)

Every time you push to `main` (or open/update a pull request), Google's **Jules** agent spins up its own cloud VM, clones your repo, reviews the latest commit for security holes, and opens a pull request with fixes. You review and merge.

Cost: ₹0 on the Jules free plan (15 tasks/day). GitHub Actions is free for public repos and has a free monthly allowance for private ones, and this job only runs for a few seconds (Jules does the heavy work on Google's side).

---

## Setup (about 5 minutes)

**1. Get a Jules API key**
- Go to [jules.google.com](https://jules.google.com), sign in with your Google account and connect GitHub.
- Give Jules access to the repo you want reviewed.
- Open **Settings → API Key** and create a key.

**2. Add the key to your repo**
- In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**
- Name: `JULES_API_KEY`
- Value: paste the key.

**3. Add the workflow**
- Copy `.github/workflows/jules-review.yml` from this repo into the same path in your repo.
- Commit and push. That push is your first review.

You'll see the run under your repo's **Actions** tab. When Jules finds something, a pull request shows up with every issue listed (file, line, why it matters, what changed).

---

## What it checks

1. Hardcoded secrets (API keys, tokens, passwords)
2. Injection (SQL, command, path traversal, unsafe eval)
3. Missing auth/access checks on routes
4. XSS, CSRF, insecure CORS, unvalidated input
5. Obvious bugs in the latest commit

Edit the `prompt:` in the workflow to change what it looks for.

---

## Limits and catches

- **Every push uses one Jules task.** Free plan: 15 tasks/day, 3 at once. Google AI Pro: 100/day. The workflow skips docs-only changes and only reviews the latest of several quick pushes.
- **Only your own pushes trigger it** (`if: github.actor == github.repository_owner`), so nobody else can burn your tasks. Remove that line for team repos, or replace it with an allowlist of usernames.
- **It reviews the latest commit**, not your whole codebase. For a one-time full audit, give Jules that task directly on jules.google.com.
- **Always review its PRs before merging.** It can be wrong.
- **If it finds a leaked key, rotate the key.** Moving it into an env variable doesn't remove it from your git history.
- **Privacy:** per Jules settings, Google doesn't train its models on content from sessions linked to private repos. Public repos have a training toggle in Jules settings. Check yours.
- GitHub only. Works best with Python and JavaScript/TypeScript.

---

## Troubleshooting

- **Workflow doesn't start:** check that the file is at `.github/workflows/jules-review.yml` and that you pushed to `main` or `master`.
- **"Unable to resolve action":** the official action is published as `google-labs-code/jules-invoke@v1` (repo: github.com/google-labs-code/jules-action). If the name changes, update the `uses:` line to match their README.
- **Auth error:** re-create the API key and check the secret name is exactly `JULES_API_KEY`.
- **No PR appeared:** that can be good news. The prompt tells Jules not to open a PR when it finds nothing. Check the run's session on jules.google.com.

---

Made by [@ran.builds](https://instagram.com/ran.builds). Free stuff for devs, with the catches.
