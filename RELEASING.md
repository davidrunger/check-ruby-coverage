# Releasing Check Ruby Coverage

Check Ruby Coverage uses semantic versioning. `VERSION` is the source of truth for the version published by the release workflow, and `CHANGELOG.md` records the user-facing changes in each release.

Release-specific tags such as `v1.2.3` never move. The compatibility tags `v1` and `v1.2` move to the newest compatible release. GitHub Releases are created only for release-specific tags, not for the moving compatibility tags.

## Repository setup

Before the first release:

1. Confirm that the repository and organization policies allow `GITHUB_TOKEN` to write repository contents. The release workflow requests only `contents: write`.
2. Enable [immutable releases](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases) in the repository settings, if available. Release-specific tags will then be locked when their GitHub Releases are published; the `v1` and `v1.2` compatibility tags remain movable because they are not associated with GitHub Releases.
3. If the repository has tag rules, allow the release workflow to create release-specific tags and update the compatibility tags.

## Preparing a release

Begin on a clean local `main` branch that matches `origin/main`. Choose the semantic version component to increment, then run one of:

```sh
bin/prepare-release major
bin/prepare-release minor
bin/prepare-release patch
```

The script:

- Requires `VERSION` to match the newest released changelog entry, or to be `0.0.0` before the first release.
- Requires `CHANGELOG.md` to have a nonempty `Unreleased` section.
- Creates a `release/vX.Y.Z` branch from `main` and configures it to track `origin/main`.
- Calculates and writes the next version to `VERSION`.
- Moves the unreleased notes into a dated section for the new version.
- Adds a new, empty `Unreleased` section.
- Creates a `[release] Prepare to release vX.Y.Z` commit containing `VERSION` and `CHANGELOG.md`.

Review the resulting commit, push the release branch, and open a pull request. Merging that pull request after CI succeeds automatically triggers the Release workflow because it changes `VERSION`. Do not add other changes to the release-preparation pull request after running the script; they would be present in the released commit but absent from its changelog entry.

For the first release, `VERSION` starts at `0.0.0`, so `bin/prepare-release major` prepares `v1.0.0`.

## Publishing a release

Merging the release-preparation pull request automatically starts the Release workflow. No version input or separate publication step is required.

The workflow reads the version from `VERSION`, verifies the corresponding changelog entry, requires the `Unreleased` section to be empty, and runs all tests. It then atomically creates the `vX.Y.Z` tag and updates the `vX` and `vX.Y` compatibility tags before creating a GitHub Release from the changelog notes. This follows [GitHub's guidance for releasing and maintaining actions](https://docs.github.com/en/actions/how-tos/create-and-publish-actions/release-and-maintain-actions).

The workflow retains its no-input manual trigger as a recovery mechanism. If the automatic run fails, rerun the failed workflow or manually run the Release workflow against `main`. It is safe to retry if tag publication succeeds but GitHub Release creation fails, and it refuses to reuse a release-specific tag that points to a different commit.

After publishing, use the workflow summary to copy the released commit SHA when pinning the action in production workflows. The README uses the convenient `v1` compatibility tag in examples, while production callers should prefer the immutable commit SHA with a version comment.
