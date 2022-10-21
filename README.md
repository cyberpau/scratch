# Create Projects from Scratch

| Stack | Directory |
|-------|-----------|
| Nuxt.JS (Vue), Firebase | [/nuxtjs-firebase](/nuxtjs-firebase/) |
| MongoDB, Express, Vue.JS, Node.JS| [/mevn](/mevn) |

<BR>

------


## General Project Guide (Node.JS)

1. Initialize the project
    ```
    npm init
    ```

2. Install dependencies (e.g. Express, Firebase)
    ```
    npm i express firebase
    ```
3. Install linters (e.g. eslint)
    ```
    npm init @eslint/config
    ```

------


## Git Flow (Feature Branch)

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



