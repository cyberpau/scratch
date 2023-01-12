# Git Branching Strategy

Previously, I'm using a single git branch strategy wherein I have my `paulo` branch together with different environment branches like `dev`, `staging`, and `prod`.

The problem with this approach is if you are working in multiple tasks and you need to push your code changes to a `main` branch, some of the pushed codes that are still on testing phase will be pushed to `main`.

```
o--- dev (working branch)
|
o--- already tested, to be pushed to prod
|
o--- still being tested
|
o---- origin/staging, origin/prod, origin/dev
|

```

In the illustration above, if you do:
```
git checkout prod
git rebase dev
git push
```

The commits `still being tested` will be pushed to prod and it may break or introduce bugs

To fix this, I have comeup with my own branching strategy (self proclaimed "own branching strategy" as I never researched or found a similar strategy online):
