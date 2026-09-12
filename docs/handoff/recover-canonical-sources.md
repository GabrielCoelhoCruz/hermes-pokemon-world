# Recover the accepted planning sources

The [handoff draft](pokemon-world-specification.md) links the accepted decisions,
but their canonical ADRs and evidence contract are still in unpublished commits
on the owner's Mac. Re-fetching the fork did not find new source branches, and
GitHub's commit API still could not resolve the reported final commit.

Run the following in the Mac checkout. The first command checks that the exact
commit exists; the second lists its documentation tree for inspection.

```bash
cd /Users/gabcruz/Projects/hermes-pokemon-world
git cat-file -e eca9f574ff72bbd3c5942eae18f7190132c6c3de^{commit}
git ls-tree -r --name-only eca9f574ff72bbd3c5942eae18f7190132c6c3de -- CONTEXT.md docs
```

If the commit exists and is the intended planning snapshot, publish that exact
commit as a separate recovery branch:

```bash
git push https://github.com/GabrielCoelhoCruz/hermes-pokemon-world.git eca9f574ff72bbd3c5942eae18f7190132c6c3de:refs/heads/recovery/world-planning
```

This publishes the committed snapshot and its required history, leaves local
uncommitted files untouched, and does not update `master`. No force push is
needed. If the remote branch already exists with different history, report
that result rather than replacing it.

If the exact commit is absent, locate the checkout/worktree containing it:
`git worktree list` and `git branch --all --contains` with the commit are useful
only in repositories where that object is available. Do not recreate the ADRs
from issue summaries. Research/prototype commits may live in separate worktrees;
their identifiers are listed in the handoff's source-recovery table.

After recovery, compare the exact documents with this draft, verify published
file hashes, and reconcile the metadata schema, operational transitions,
migration rules, and 54 acceptance rows. Only then can PR #13 be evaluated as a
complete planning handoff and issue #1 be completed.
