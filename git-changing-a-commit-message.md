# Changing a commit message

This document summarizes [Changing a commit message](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message) from GitHub Docs.

If a commit message is unclear, wrong, or contains sensitive information, you can change it locally and update the remote. Amending changes the commit ID (SHA), because the message is part of the commit.

---

## Most recent commit (not pushed)

If the commit exists only locally:

1. Open a terminal in the repository.
2. Run `git commit --amend` and press Enter.
3. In your editor, edit the message, save, and close.
   - You can add a [co-author](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors) or [commit on behalf of an organization](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-on-behalf-of-an-organization) using trailers.

4. Push when ready; the new commit replaces the old one locally.

**Tip:** Set your preferred editor with Git’s `core.editor`. See [Basic Client Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration#_basic_client_configuration) in the Git book.

---

## Most recent commit (already pushed)

1. Amend locally as above (`git commit --amend`).
2. Update the remote with a safe force push:

   ```shell
   git push --force-with-lease origin YOUR-BRANCH
   ```

**Warning:** Force pushing rewrites history. Others who cloned the repo may need to fix their local history. See [Recovering from upstream rebase](https://git-scm.com/docs/git-rebase#_recovering_from_upstream_rebase) in the Git manual.

---

## Older or multiple commits

Use interactive rebase, then force push.

1. In the repository directory, run:

   ```shell
   git rebase -i HEAD~n
   ```

   Replace `n` with how many recent commits to list (e.g. `HEAD~3` for the last three).

2. In the editor, change `pick` to `reword` (or `r`) for each commit whose message you want to change.

   Example rebase todo list:

   ```text
   pick e499d89 Delete CNAME
   reword 0c39034 Better README
   reword f7fde4a Change the commit message but push the same commit.
   ```

3. Save and close the todo file.

4. For each `reword` commit, Git opens an editor; set the new message, save, close.

5. Push with force (coordinate with your team):

   ```shell
   git push --force origin YOUR-BRANCH
   ```

Prefer `--force-with-lease` over `--force` when you want to avoid overwriting others’ work without noticing.

**Note:** Every amended commit gets a new ID. Any descendant commit on that branch also gets a new ID.

---

## Sensitive data in a commit message

Force-pushing an amended commit does not guarantee removal of the old commit from GitHub’s servers; it may still be reachable by SHA for a time. For purging sensitive data from the remote, use [GitHub Support](https://support.github.com) with the old commit ID as described in the official docs.

---

*Source: [GitHub Docs — Changing a commit message](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/changing-a-commit-message) (accessed for this summary).*
