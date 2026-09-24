# Git Workflow Task

**Repository Link:** https://github.com/poojapc754-ui/construction-technect-devops-test  
**Branch Name:** devops-test

---

## Steps Performed

1. Created repository: construction-technect-devops-test
   git init
   (or created directly on GitHub)

2. Added README.md
   git add README.md
   git commit -m "Initial commit with README"

3. Created a new branch
   git checkout -b devops-test

4. Made a change to README.md
   (edited file)

5. Committed the change
   git add README.md
   git commit -m "Update README with project details"

6. Pushed the branch to GitHub
   git push origin devops-test

---

## How I Would Create a Pull Request

1. Go to the repository on GitHub.
2. GitHub shows a banner suggesting to compare & create a pull request for the recently pushed branch.
3. Click "Compare & pull request".
4. Select the base branch (e.g. main) and compare branch (devops-test).
5. Add a title and description explaining the changes.
6. Click "Create pull request".
7. Once reviewed and approved, click "Merge pull request".

---

## How I Would Handle a Merge Conflict

If another developer modified the same file:

1. Pull the latest changes: git pull origin main
2. Git will flag the conflicting file(s) with conflict markers:
   <<<<<<< HEAD
   (my changes)
   =======
   (their changes)
   >>>>>>> branch-name
3. Open the file, manually review both changes, and decide what to keep (or combine both).
4. Remove the conflict markers once resolved.
5. Stage the resolved file: git add <filename>
6. Commit the merge: git commit -m "Resolve merge conflict in README.md"
7. Push the resolved branch: git push origin devops-test
