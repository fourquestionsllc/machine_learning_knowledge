To merge a branch from a Bitbucket repository into a branch in a GitHub repository using Git, follow these steps:

---

### Steps to Merge a Branch from Bitbucket to GitHub

#### 1. **Ensure Both Repositories Are Cloned**
You should have cloned both repositories:
- **Bitbucket repo**: Assume it's cloned to a directory, e.g., `bitbucket-repo/`.
- **GitHub repo**: Assume it's cloned to a directory, e.g., `github-repo/`.

#### 2. **Add the Bitbucket Repository as a Remote in GitHub Repo**
Navigate to the directory of your **GitHub repo**:
```bash
cd github-repo
```

Add the Bitbucket repository as a remote (you can name it `bitbucket` or any alias):
```bash
git remote add bitbucket <bitbucket-repo-url>
```

Verify the remotes:
```bash
git remote -v
```

You should see something like:
```
origin    <github-repo-url> (fetch)
origin    <github-repo-url> (push)
bitbucket <bitbucket-repo-url> (fetch)
bitbucket <bitbucket-repo-url> (push)
```

#### 3. **Fetch the Branch from Bitbucket**
Fetch the branch you want to merge from the Bitbucket repository:
```bash
git fetch bitbucket <branch-name>
```

For example:
```bash
git fetch bitbucket feature-branch
```

#### 4. **Check Out the Target Branch in the GitHub Repository**
Switch to the branch in the GitHub repository where you want to merge the Bitbucket branch:
```bash
git checkout <github-branch-name>
```

For example:
```bash
git checkout main
```

#### 5. **Merge the Bitbucket Branch**
Merge the branch fetched from the Bitbucket repository:
```bash
git merge bitbucket/<branch-name>
```

For example:
```bash
git merge bitbucket/feature-branch
```

Resolve any conflicts if they arise during the merge process.

#### 6. **Push the Changes to GitHub**
After the merge, push the changes to the GitHub repository:
```bash
git push origin <github-branch-name>
```

For example:
```bash
git push origin main
```

---

### Optional: Clean Up
- If you no longer need the Bitbucket remote, you can remove it:
  ```bash
  git remote remove bitbucket
  ```

---

### Summary
1. Add the Bitbucket repo as a remote in the GitHub repo.
2. Fetch the branch from the Bitbucket repo.
3. Merge the fetched branch into the desired branch in the GitHub repo.
4. Push the merged changes to GitHub.
