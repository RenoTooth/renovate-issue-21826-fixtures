# Renovate #21826 reproduction

Three pinned dependencies share pnpm-lock.yaml. Renovate groups is-number (A), is-odd (B), and is-even (C), with fixed target versions for repeatability. Automerge is disabled.

## Group shrink

1. Run Renovate from the fix checkout against this repository with repository caching enabled. Initial config should create one grouped PR/MR upgrading all three packages. Do not merge it.
2. Copy scenarios/shrunk.json over renovate.json and commit on main. This groups only A+B; C can receive a separate PR/MR.
3. Run with repository caching disabled (`RENOVATE_REPOSITORY_CACHE=disabled`) to reproduce the missing-fingerprint condition. The original bug leaves C upgraded in the grouped branch while its description lists only A+B.
4. With the fix, the grouped branch must contain A=7.0.0, B=3.0.1, C=0.1.0 in both package.json and pnpm-lock.yaml. The description must list only A+B. A separate C branch may contain C=1.0.0.
5. Run again with caching enabled: an unchanged group should not receive another commit. Compare branch SHAs.

## Manual edits and unpend

1. Reset main to the initial scenario and create the initial grouped branch again.
2. Add manual.txt on the grouped branch, committed by your human identity, and push it. Record its SHA.
3. Copy scenarios/pending.json over renovate.json on main and run Renovate. The updates should appear as pending in the Dependency Dashboard.
4. Check that branch's unpend box and run again. The fixed version must preserve manual.txt and the branch SHA, return the branch to edited/blocked status, and retain the warning on the PR/MR.
5. An explicit rebase checkbox or dashboard rebase request may discard the custom commit. Unpend alone must not.

Also test rebaseWhen=never: automatic reconciliation stays suppressed until an explicit rebase request. Run upstream and the fix against separate fresh fixture state; never merge generated update PRs/MRs during comparison.

## Runner

Use your existing authenticated Renovate runner, selecting this repository only. For GitHub use platform=github; for GitLab use platform=gitlab. Keep tokens in the runner environment. To run the patched source from its checkout, use `pnpm start --platform=<platform> --repositories=<owner/repo>` with the runner's normal credentials and execution settings. The repository does not deploy or run CI jobs.

Fixture creation does not itself validate the bot's remote behavior. Compare actual Git files, shared lockfile, branch SHA, and PR/MR description after each run.
