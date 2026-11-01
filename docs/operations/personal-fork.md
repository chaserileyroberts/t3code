# Maintaining the image-preview fork

The default branch is [`chase/image-previews`](https://github.com/chaserileyroberts/t3code/tree/chase/image-previews), based on upstream **v0.1.6**. The fork's `main` branch remains an upstream reference.

Two separate source changes are carried on this branch:

- Codex `imageView` completions become image activity entries. Existing desktop, web, and native mobile clients can render them.
- Desktop and host-served web chats open viewed-image previews by default and keep them outside collapsed activity groups and completed-turn folds. Individual previews can still be collapsed. Native mobile expansion behavior is unchanged.

The commits include adapter replay tests and browser interaction tests. Old image events that the server already dropped are not backfilled.

## Remotes

`origin` is `https://github.com/chaserileyroberts/t3code.git`; `upstream` is `https://github.com/pingdotgg/t3code.git`. After a fresh clone, add the upstream remote and select the maintained branch:

```sh
git remote add upstream https://github.com/pingdotgg/t3code.git
git switch chase/image-previews
git config remote.pushDefault origin
```

Follow the [development setup](development.md#first-checkout) for the Node/package-manager versions and dependencies required by the chosen release.

## Move to a new stable release

Check [upstream releases](https://github.com/pingdotgg/t3code/releases). From a clean checkout of `chase/image-previews`, fetch upstream tags and merge the chosen stable tag:

```sh
git fetch upstream --tags
git merge vX.Y.Z
```

Replace `vX.Y.Z` with the actual release tag. Use a merge so the published branch remains fast-forwardable. Resolve conflicts in the source files; if upstream has implemented one of these changes, retain its implementation and remove the redundant fork change. Do not replace the branch with upstream's `main` or the fork's Sync button, which can discard or mix release-based customizations.

Install the new release's dependencies, then check the fork changes against that tag:

```sh
vp install
vp run check:changed vX.Y.Z
```

For a focused image regression check:

```sh
vp test run apps/server/src/orchestration-v2/Adapters/CodexAdapterV2.test.ts -t "projects completed image views"
vp test run apps/web/src/components/chat/MessagesTimeline.browser.test.tsx -t "opens viewed images"
```

Review the merged changes, update the base release recorded above, and push:

```sh
git push origin chase/image-previews
```

## Running and distributing the fork

Keeping these commits in GitHub does not switch an existing official T3 installation to this fork. Its official updater can still replace local patches. Use a build of this branch to carry the changes forward; keep development state isolated as described in `AGENTS.md`.

The repository's `dist:desktop:win:x64` task builds a Windows installer. The packaging script accepts `--build-version` for the compiled version and honors `T3CODE_DESKTOP_UPDATE_REPOSITORY=chaserileyroberts/t3code` when building a fork that will publish its own update artifacts. Configure that feed only when maintaining actual releases in this fork; creating the source fork alone does not provide an update feed.

Machine-specific ASAR patches, installers, backups, transcripts, and local application data are not part of this source branch. No upstream issue or pull request is required to maintain the fork.
