# Development environment, command line, and Git

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [web fundamentals](../01-web-and-browser-fundamentals/tutorial.md). Have a browser, editor, Git, and an installed Python 3 interpreter for the optional server. This tutorial builds a reproducible local workflow: organize files, run commands in the right directory, record a change, inspect differences, and recover safely from ordinary mistakes.

## What belongs in a development environment

An editor changes source files. A terminal runs programs. A shell interprets command syntax. Git records snapshots. A runtime executes a language outside the browser, while a package manager resolves and installs dependencies. A development server serves local files and may rebuild code. These jobs can be bundled in one application, but keeping the responsibilities separate makes failures easier to locate.

Every command has a working directory, arguments, environment, output, and exit status. Relative paths start from the working directory. Quote paths containing spaces. The commands below work in PowerShell; avoid pasting Unix-only shell substitutions into a Windows terminal. Read commands before executing them, especially recursive deletion, hard resets, remote scripts, and force pushes.

For a JavaScript project, record the supported Node.js and package-manager versions in its documentation and version configuration. Commit a package manifest and one appropriate lockfile. Installing from a lockfile produces a more repeatable dependency graph than resolving open ranges on every machine. Lockfiles do not guarantee identical operating systems, native binaries, environment variables, or registry availability. Keep credentials out of source and out of browser-delivered environment variables: anything bundled into frontend code is observable by visitors.

Git has a working tree, staging area, and commit history. Editing changes the working tree; `git add` selects content for the next snapshot; `git commit` records that staged snapshot. A branch is a movable reference to a commit, not a separate copied directory. The official [Git basics chapter](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics) develops this snapshot model.

## Worked tutorial: create and record a page

1. Open PowerShell in a location where you can create a new folder. Run each command separately:

```powershell
mkdir frontend-lab
cd frontend-lab
git --version
python --version
git init
```

2. In your editor, create `index.html`:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>My learning log</title>
<main>
  <h1>My learning log</h1>
  <p>I can inspect a browser request.</p>
</main>
</html>
```

3. Create `.gitignore` with the following lines. Ignoring a path does not remove files already tracked, and it does not erase earlier commits containing secrets.

```gitignore
node_modules/
dist/
.env
.env.*
!.env.example
```

4. Inspect and record the exact changes:

```powershell
git status
git add index.html .gitignore
git diff --cached
git commit -m "Add learning log page"
```

If Git asks for an identity, set your own name and email for this repository with `git config user.name "Your Name"` and `git config user.email "you@example.com"`, replacing both values. A commit email becomes part of history; use an appropriate address. This tutorial never pushes to a remote.

5. Start the static server and visit `http://127.0.0.1:8000`:

```powershell
python -m http.server 8000 --bind 127.0.0.1
```

Stop it with Ctrl+C. Create a branch, change the paragraph in the editor, and inspect the difference:

```powershell
git switch -c improve-learning-log
git diff
git add index.html
git commit -m "Describe the next learning objective"
git log --oneline -3
git status
```

The expected result is two commits and a clean working tree on `improve-learning-log`. The page remains runnable without a package registry. If `python` is unavailable on Windows but the Python launcher is installed, use `py -3 -m http.server 8000 --bind 127.0.0.1`.

## Recovery, collaboration, and tradeoffs

`git restore --staged index.html` removes a file's changes from staging while keeping the working copy. `git restore index.html` discards unstaged edits to that tracked file; inspect the diff first because those edits may not exist in history. `git revert <commit>` creates a new commit reversing an earlier change and is often appropriate for shared history. A reset or rebase rewrites references/history and requires more care once others depend on it.

When merging causes a conflict, inspect both intended behaviors, edit the file into one correct result, remove conflict markers, run relevant checks, and commit the resolution. Do not simply select one side because the editor offers a convenient button. Small coherent commits and descriptive pull requests make review and recovery easier.

Use an editor formatter to reduce formatting disputes, a linter for suspicious patterns, type checking for contracts, and tests for behavior. They complement one another. Agree on line endings and encoding, especially across operating systems. An `.editorconfig` can express indentation and newline preferences; `.gitattributes` can normalize text handling. Never treat passing automated checks as proof that the visual interface is usable.

## Exercises and acceptance criteria

- Stage one file while leaving another unstaged. `git diff` and `git diff --cached` must show the intended separate changes.
- Add a `README.md` with setup, run, and stop instructions. Another person should run the page from those instructions alone.
- Introduce and revert a harmless paragraph change using `git revert`. History must retain both the original change and its reversal.
- Document one environment failure, including the command, directory, error, and fix. Do not include secrets.

## References

- [Git: reference manual](https://git-scm.com/docs)
- [Git: recording changes](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
- [Python: HTTP server](https://docs.python.org/3/library/http.server.html)
- [npm: clean installation with npm ci](https://docs.npmjs.com/cli/commands/npm-ci)
