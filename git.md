# Git Configuration

## Line Breaks

If you’re working using Windows, you would probably encounter some linebreak linting issues. ESLint requires linebreak to be LF but once you push it to a git repository and pull it to your Windows system, it becomes CRLF. The manual and repetitive way to fix this is to run eslint --fix. To fix this permanently, run the following:

```
git config --global core.autocrlf false
```

Source: [Git replacing LF with CRLF](https://stackoverflow.com/questions/1967370/git-replacing-lf-with-crlf)

## Set Username and Email

If you have used git before, probably you are using your personal account in the git config. To update this, run the following inside your work repository:

```
git config user.name "FIRST_NAME LAST_NAME"

git config user.email "MY_NAME@example.com"
```


# Git Workflow Strategies

## Feature Branch

This is a short version of the [Git Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) by Atlassian.

1. Start off with a clean and latest state. Assuming the main branch is on `main`, do the following:

    ```
    git checkout main // this will switch the branch to the main branch
    git fetch origin // pull the latest commits
    git reset --hard origin/main // resets the local copy of main
    ```

2. Create a feature branch. The `-b` flag tells git to create a branch if it doesn't already exist.

    ```
    git checkout -b <new-feature>
    ```

3. Update, add, commit and push changes

    ```
    git status
    git add <some-files or .>
    git commit -m "<your-message>"
    ```

4. Push feature branch to remote. 

    ```
    git push -u origin <new-feature>
    ```

5. Once approved by code-review, you can now merge your changes to the `main` branch.

    ```
    git checkout main
    git pull
    git pull origin <new-feature>
    git push
    ```

6. Once work is completed on a feature, it is often recommended to delete the branch.

    Delete a remote branch:
     `git push -d origin <branch>`
    Example
     `git push -d origin dev-backend-users `

    Delete a local branch:
     `git branch -d <branch>`



## Rebase Strategy

1. From the main branch, create your own local branch. You will only do this ONE TIME. This will be the only branch you will be commiting on.

```
git pull
git checkout -b paulo
```

2. Evertime you need to make code changes, first ensure that the main branch is latest.

```
git checkout main
git pull
```

3. Go to dev branch and  do some code changes. Commit and push to remote branch. 
```
git push -u origin paulo
```

4. To ensure that your dev branch `paulo` is still up to date, go to main branch and pull the latest changes. Then go back to dev branch
```
git checkout main
git pull
git checkout paulo
git rebase main
```

5. Go back to main branch, rebase from your dev branch and then push the changes
```
git checkout main
git rebase paulo
git push
```

6. Optional, to ensure that your local dev branch `paulo` is the same as remote branch `origin/paulo`:
```
git checkout paulo
git pull
git push -u origin paulo
```
