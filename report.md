# Lab 03 — Git in Depth: Branching, Rebasing and Finding Regressions

## 1. Git Objects

The initial commit was `a142011` (`Add Lab 02 build pipeline`).

- `81f0e2e4f8967d23ca76457e22622c25576cfe7b` — **tree object** referenced by the initial commit; it represents the repository root tree.
- `d84960dd2ae67969731b9a479cf884ff3c344e09` — **tree object** representing the `src/` directory.
- `5f32579b39b9805a9705ffe6e39a04f0455b134a` — **blob object** containing `src/services.txt`.

The `services.txt` file contains the service names used in the project.

## 2. Main Branch History

```text
*   6fdd278 (HEAD -> main, tag: v1.0.0) Merge feature/docs
|\
| * 72fc8e1 (feature/docs) Document the Makefile targets inline
|/
* 069cf47 Add a README
*   13547b4 Merge feature/docs and resolve report conflict
|\
| * 1459a8c Update report on feature branch
* | 14666ca Update report on main branch
|/
* be23c25 Correct the contents of report.txt
* de34c2e Add report.txt to the build inputs
```

The final main branch contains two merge commits and the `v1.0.0` annotated tag. The temporary `wip`, `fix`, and `more` commits were squashed during interactive rebase and do not remain in the final main history.

## 3. Rebase and Commit Hash Change

The feature branch originally contained three temporary commits: `wip`, `fix`, and `more`. These were squashed using interactive rebase into the single commit `Document the Makefile targets inline`.

After `main` moved forward with the `Add a README` commit, the feature branch was rebased onto the updated `main`. The feature commit hash changed from `e5b1317` to `72fc8e1`.

The hash changed because a Git commit includes its parent commit information. Rebasing changes the parent of the commit, so Git creates a new commit with a different hash even though the logical changes are the same.

Both `git commit --amend` and `git rebase` produce new hashes for the same reason: a commit's name is the hash of its contents (tree, parent, author, message). Amend changes the message and rebase changes the parent, so Git must create a new commit object instead of editing the old one.

## 4. Conflict Resolution

During Task 3, a conflict occurred in `src/report.txt`.

The main branch wanted:

```text
Main branch documentation
```

while the feature branch wanted:

```text
Feature branch documentation
```

The conflict was deliberately resolved as:

```text
Main branch documentation
Feature branch documentation
```

This result is neither side verbatim because it combines the content from both branches into one final file.

The conflict was in src/report.txt (not in the Makefile as the manual's example): main wanted "Main branch documentation" and the feature branch wanted "Feature branch documentation". Both lines were valuable and Git cannot guess which one should win, so I combined them into one file. This is why my resolution is neither side verbatim.

The first merge (13547b4) was a plain `git merge`; because of the conflict Git had to create a two-parent merge commit anyway. The second merge (6fdd278) used `--no-ff`.

## 5. Tagging and CI

An annotated tag named `v1.0.0` was created on the final merge commit `6fdd278`.

The tag message was:

```text
Lab 03 release
```

The final CI check was executed using:

```bash
make ci
```

The check passed successfully. Four source files were found, all four were built, and the test reported:

```text
TEST PASSED
```

## 6. Finding the Regression with Git Bisect

The regression demo contained 16 commits. The test script failed on the current bad version, so `git bisect` was used to locate the first bad commit.

The command used was:

```bash
git bisect start HEAD $(git rev-list --max-parents=0 HEAD)
git bisect run ./test-build.sh
```

Git tested four commits and identified:

```text
2d389cb76ad43436966cfd3078939815c401214d
```

as the first bad commit.

Its commit message was:

```text
Speed up the transform step
```

Bisect does a binary search: each test halves the remaining candidates (16 → 8 → 4 → 2 → 1), so it needs log2(16) = 4 tests instead of up to 16.

Afterwards, `git bisect reset` was used and the generated test files were removed.

## 7. Final Status

The final repository has a clean working tree, the required `v1.0.0` annotated tag, the squashed feature history, two `--no-ff` merge commits, and a passing `make ci` check.
I used my own Lab 02 Makefile and did not use the provided fallback/ folder.
