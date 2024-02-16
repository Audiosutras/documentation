---
category: Git
---
# Code Versioning & Rolling Back Main Branch To A Prior State

## Code Versioning

From the command line type:
```bash
# make sure you are on the main branch of your project
$ git tag
v0.1.2
v0.1.3
v0.1.4
# incrementing up, other git tags
# ...
```
The command above lists out the git `tags` associated with the git repository. Each `tag` represents a snapshot of the codebase at a given point in time. This is both important and useful for us as developers because when something goes wrong we can use a `tag` to restore the codebase to a state when our application logic was in a good working condition.

To create a new tag for the production branch of our application run the following:
```bash
$ git checkout main
$ git tag vX.X.X # where X.X.X is a version number
```
Make sure our version number is incrementally higher than the preceding `tags`. For this example, `v0.1.4` is our latest tag so our new tag will be `v0.1.5` (or `0.2.0`, its up to you).

Now lets create our new tag `v0.1.5` and push it to the repository:
```bash
$ git tag v0.1.5
$ git push origin v0.1.5
```
Our tag is now associated with the remote repository. To learn more about git tags see [here](https://git-scm.com/book/en/v2/Git-Basics-Tagging)

## Rolling Back Main Branch To A Prior State

If you are reading this section then you know you messed up somewhere. We do not know yet where and we want to fix the state of our codebase. No worries, let's [roll back our codebase to a prior tag](https://stackoverflow.com/questions/6872223/how-do-i-revert-master-branch-to-a-tag-in-git).

Sticking with the example from [Codebase Versioning](#code-versioning) say we want to roll our codebase back to tag `v0.1.4` from an unknown amount of commits past `v0.1.5` because we know for certain that `v0.1.4` is a point in time when our codebase was in good health. We can run the following:
```bash
$ git checkout v0.1.4
$ git diff main > ~/diff.patch
$ git checkout main
$ cat ~/diff.patch | git apply
$ git commit -m "Roll back to v0.1.4"
$ git push origin main
```
This will roll back the `main` branch to `v0.1.4` while keeping commit history intact.

