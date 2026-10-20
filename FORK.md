# byvict/stats fork

This fork carries a few commits on top of [exelban/stats](https://github.com/exelban/stats) so the Disk mini widget can show several disks (for example "SSD 66%/8%"). There is no Xcode on the owner's Mac. GitHub Actions builds the app and the owner installs it by hand, following [COMO-ATUALIZAR.md](COMO-ATUALIZAR.md). Never open pull requests from this fork to exelban/stats.

## Branches

- `custom` is the default branch. It is exelban/stats `master` with our commits rebased on top, and every installable build comes from it.
- `master` is the fork's original copy of upstream. Nothing uses it and nothing rewrites it.
- Any other branch is a work branch. Start it from `custom`. The `fork-build` workflow builds every push to it.

`custom` is the default branch because GitHub runs scheduled workflows only from the default branch. Our commits stay a short linear series on top of upstream, listed by `git log upstream/master..custom`, which keeps them easy to review and to replay on each upstream release.

## Weekly sync

`.github/workflows/fork-sync.yaml` runs every Monday at 09:17 UTC, the morning after upstream's usual Sunday release. It can also be started by hand from Actions, fork-sync, Run workflow. It does the following.

1. It rebases `custom` onto exelban/stats `master`. On a conflict it aborts, names the conflicting files in the run annotations, and fails. GitHub then emails the owner.
2. It force-pushes `custom` with a lease on the commit it started from, so a push made while it ran is never overwritten.
3. It builds and uploads `Stats-<commit sha>` when no unexpired artifact with that name exists yet. A run with nothing new upstream ends green without a build. A run after a failed build builds again. Artifacts are kept for 90 days.

The build itself lives in `.github/actions/build-app`, shared with `fork-build`. It archives without a signing identity, ad-hoc signs the app, and zips it with `ditto`.

The install how-to downloads the artifact of the current `custom` commit. After pushing to `custom` by hand, run fork-sync once so that commit gets a build.

### Fixing a rebase conflict

```bash
git fetch upstream
git switch custom
git rebase upstream/master
```

Resolve each conflict, then `git rebase --continue`. For a large resolution, push it to a work branch first so `fork-build` proves it compiles. Finish with `git push --force-with-lease origin custom` and run fork-sync by hand to get a build. Upstream edits `Modules/Disk/main.swift` about every two weeks, so expect this now and then.

## Runner capacity

macOS arm64 runners are sometimes short on capacity, and a job can fail with "The job was not acquired by Runner of type hosted". The `build` job has `continue-on-error: true`, and `build-retry` runs the same steps once more when `build` did not finish. The run fails only when both attempts fail. Then use "Re-run failed jobs" on the run page, or wait for the next Monday. That run sees there is no artifact for the current `custom` commit and builds it.

## Push credential

The default `GITHUB_TOKEN` cannot push commits that change files under `.github/workflows`, and no `permissions:` key grants that. Upstream edits its workflows often (six commits between 2026-09-28 and 2026-10-18). When that happens, a push with `GITHUB_TOKEN` is rejected and the run fails with a "Push rejected" annotation.

fork-sync therefore checks out with the `FORK_SYNC_SSH_KEY` secret when it exists. That secret holds the private half of a deploy key with write access to this repository only. It was added on 2026-10-20. Unlike a `GITHUB_TOKEN` push, a deploy-key push triggers upstream's own push workflows on `custom` (Linter, i18n check and SMC tests, each only when its paths changed). A failure there points at upstream code, not at the fork. Without the secret, fork-sync falls back to `GITHUB_TOKEN`, which works for every week in which upstream did not touch its workflows.

To replace the key, delete the old one and run the following. The owner runs it, or approves an agent running it.

```bash
ssh-keygen -t ed25519 -N '' -C fork-sync -f /tmp/fork-sync
gh repo deploy-key add /tmp/fork-sync.pub -R byvict/stats --allow-write -t fork-sync
gh secret set FORK_SYNC_SSH_KEY -R byvict/stats < /tmp/fork-sync
rm /tmp/fork-sync /tmp/fork-sync.pub
```

To revoke it, delete the "fork-sync" key under Settings, Deploy keys, and the secret under Settings, Secrets and variables, Actions.

## Scheduled workflow caveats

- GitHub disables scheduled workflows by default when a public repository is forked. The fork's Actions were enabled before fork-sync was added, and the workflows API lists fork-sync as `active`. If the Actions tab ever shows fork-sync as disabled, open it and click "Enable workflow".
- GitHub also disables scheduled workflows in a public repository after 60 days without repository activity. GitHub does not document whether fork-sync's own pushes count as activity. After a long upstream pause, check the Actions tab and click "Enable workflow" if needed.
- The app's own updater would replace this build with the official release. The `update-interval` preference must stay `Never`, see [COMO-ATUALIZAR.md](COMO-ATUALIZAR.md). The app's default when the preference is missing is a silent update.
