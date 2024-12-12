When you encounter conflicts during a `git merge`, Git will stop the merge process and allow you to resolve the conflicts manually. Here's how to handle them step by step:

---

### Steps to Resolve Conflicts During `git merge`

#### 1. **Check the Status**
After encountering conflicts, run:
```bash
git status
```
This will show a list of files with conflicts, marked as **both modified**.

#### 2. **Open the Conflicted Files**
Open the files with conflicts in your code editor. Conflicted sections will look like this:
```plaintext
<<<<<<< HEAD
Code from the current branch
=======
Code from the branch being merged
>>>>>>> bitbucket/branch-name
```

#### 3. **Resolve the Conflicts**
Edit the conflicted sections to decide what the final code should be:
- Keep one side.
- Combine both changes.
- Make custom adjustments as needed.

For example:
```plaintext
Final resolved code here
```

Remove the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) after resolving.

#### 4. **Mark Resolved Files**
After resolving conflicts in a file, mark it as resolved by staging it:
```bash
git add <file>
```

For example:
```bash
git add conflict-file.txt
```

#### 5. **Continue the Merge**
After resolving all conflicts and staging the resolved files, complete the merge:
```bash
git commit
```
Git will automatically generate a default merge commit message, but you can edit it if needed.

#### 6. **Push the Changes**
Once the merge is complete, push the changes to the remote repository:
```bash
git push origin <branch-name>
```

---

### Tips for Resolving Conflicts
1. **Use a Merge Tool**:
   You can use a visual merge tool to make resolving conflicts easier:
   ```bash
   git mergetool
   ```
   Common tools include:
   - [VS Code](https://code.visualstudio.com/)
   - [Sublime Merge](https://www.sublimemerge.com/)
   - [KDiff3](http://kdiff3.sourceforge.net/)
   - [P4Merge](https://www.perforce.com/products/helix-core-apps/merge-diff-tool-p4merge)

2. **Abort the Merge if Needed**:
   If you want to stop the merge and reset the branch to its pre-merge state:
   ```bash
   git merge --abort
   ```

3. **Collaborate**:
   If you're unsure about how to resolve a conflict, discuss it with your team or the person who worked on the conflicting changes.

---

### Example
Here’s how it might look in practice:
1. Merge the branch:
   ```bash
   git merge bitbucket/feature-branch
   ```
   Conflict detected:
   ```plaintext
   Auto-merging file.txt
   CONFLICT (content): Merge conflict in file.txt
   Automatic merge failed; fix conflicts and then commit the result.
   ```

2. Resolve the conflict in `file.txt`:
   Before:
   ```plaintext
   <<<<<<< HEAD
   Current branch code
   =======
   Code from bitbucket branch
   >>>>>>> bitbucket/feature-branch
   ```
   After:
   ```plaintext
   Final combined code
   ```

3. Stage the file:
   ```bash
   git add file.txt
   ```

4. Commit the merge:
   ```bash
   git commit
   ```

5. Push the changes:
   ```bash
   git push origin main
   ```
