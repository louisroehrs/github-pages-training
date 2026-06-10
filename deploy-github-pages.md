---
description: Deploy the current project to GitHub Pages. Pass an optional repo name as $ARGUMENTS.
---

Deploy the current project to GitHub Pages. Use $ARGUMENTS as the repo name if provided, otherwise derive a name from the current directory.

Work through these steps in order. Stop and explain clearly if any step fails.

## Step 1 — Check prerequisites

Run `git --version`. If git is not installed, tell the user to install it from git-scm.com and stop.

Run `gh --version`. If gh is not installed, tell the user to install it from cli.github.com and stop.

Run `gh auth status`. If not authenticated, run `gh auth login` and wait for the user to complete the browser flow before continuing.

## Step 2 — Determine the repo name

If $ARGUMENTS is non-empty, use it as the repo name.
Otherwise, use the name of the current working directory (basename only, lowercased, spaces replaced with hyphens).

Store this as REPO_NAME for the steps below.

## Step 3 — Initialize git if needed

Check whether a `.git` directory exists in the current folder.

If it does NOT exist:
- Run `git init`
- Run `git add .`
- Run `git commit -m "Initial commit"`

If it already exists, check if there are uncommitted changes with `git status --porcelain`. If there are, stage and commit them with a sensible message before continuing.

## Step 4 — Create GitHub repo and push

Run `gh repo view 2>/dev/null` to check if a GitHub remote already exists for this directory.

If there is NO existing GitHub remote:
- Run: `gh repo create REPO_NAME --public --source=. --remote=origin --push`
- This creates the repo, sets the remote, and pushes in one command.
- Skip to Step 5.

If a remote already exists:
- Ensure the current branch is named `main` (`git branch -M main`).
- Run `git push -u origin main`.

## Step 5 — Enable GitHub Pages

Determine the GitHub owner and repo from `gh repo view --json owner,name --jq '[.owner.login,.name] | join("/")'`.

Call the Pages API to enable deployment from the main branch root:

```
gh api -X POST /repos/OWNER/REPO/pages \
  -f source[branch]=main \
  -f source[path]=/
```

If the response is a 409 (Pages already configured), that is fine — continue.

## Step 6 — Confirm the live URL

Poll `gh api /repos/OWNER/REPO/pages --jq '.html_url'` every 5 seconds, up to 12 attempts (60 seconds total), until it returns a non-null URL.

When it returns:
- Print: `✓ Live at <URL>`
- Remind the user that the first deploy takes about 60 seconds and they can watch progress in the Actions tab on GitHub.

If it times out after 60 seconds, print the expected URL in the format `https://OWNER.github.io/REPO` and tell the user to check the Actions tab.
---
description: Deploy the current project to GitHub Pages
---

Deploy this project to GitHub Pages.
Repo name (optional): $ARGUMENTS

1. Verify git is installed. If not, tell the user and stop.
2. Verify gh is installed and authenticated (gh auth status).
   If not authenticated, run gh auth login and wait.
3. If no .git folder exists: git init, git add ., git commit.
4. If no remote 'origin': gh repo create $ARGUMENTS --public
   --source=. --remote=origin --push, then skip step 5.
5. Otherwise: git push -u origin main
6. Enable Pages: gh api -X POST /repos/{owner}/{repo}/pages
   -f source[branch]=main -f source[path]=/ (ignore 409).
7. Poll gh api /repos/{owner}/{repo}/pages --jq .html_url
   until it returns a URL. Print: 'Live at <url>'
