To completely delete a Git branch, both locally and remotely, you can follow these steps:

### 1. **Delete a Local Git Branch**
To delete a branch **locally**, you can use the following command:

#### a. **Delete the Local Branch**
First, ensure you're not currently on the branch you want to delete. You cannot delete the branch you're currently on.

- Switch to a different branch (e.g., `main` or `master`):
  ```bash
  git checkout main
  ```

- Now, to delete the local branch, use the following command:
  ```bash
  git branch -d <branch-name>
  ```

  - The `-d` flag (which stands for "delete") will delete the branch only if it has already been merged into your current branch or another branch (usually `main` or `master`).
  - If you are **absolutely sure** you want to delete the branch (even if it hasn't been merged), use the `-D` flag:
    ```bash
    git branch -D <branch-name>
    ```

#### b. **Verify Deletion**
To check if the branch has been deleted locally, you can list all branches:
```bash
git branch
```

### 2. **Delete a Remote Git Branch**
If the branch you want to delete exists on a remote repository (e.g., GitHub, GitLab, Bitbucket), you can delete it as follows:

#### a. **Delete the Remote Branch**
To delete the remote branch, use the following command:
```bash
git push origin --delete <branch-name>
```

- This command tells Git to push the deletion of the branch to the remote (`origin` in this case).

#### b. **Verify Deletion on Remote**
To ensure the branch was deleted remotely, you can list the remote branches:
```bash
git branch -r
```

Alternatively, you can fetch the latest branch list and check:
```bash
git fetch -p
```
The `-p` (or `--prune`) option will remove any stale references to remote branches that no longer exist.

### 3. **Clean Up Stale References Locally (Optional)**
Sometimes, after deleting a remote branch, local references to it may still exist. You can remove these "stale" references using the following command:
```bash
git remote prune origin
```

This command will clean up any remote-tracking branches that no longer exist on the remote.

---

### Summary of Commands:
- **Delete a local branch:**
  - `git branch -d <branch-name>` (safe delete)
  - `git branch -D <branch-name>` (force delete)
  
- **Delete a remote branch:**
  - `git push origin --delete <branch-name>`
  
- **Clean up stale references:**
  - `git remote prune origin`

By following these steps, you can completely delete a Git branch both locally and remotely.
