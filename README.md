# Shared GitHub Actions workflows

This repository provides reusable workflows for MontFerret projects.

## Notify the website after publishing a release

`.github/workflows/notify-website-release.yml` announces a successfully published release to `MontFerret/montferret.github.io`. The website owns version validation, `data/versions.yaml`, and pull request creation; callers only provide release metadata.

Call the reusable workflow only after the job that actually publishes the release succeeds:

```yaml
notify-website:
  needs: goreleaser
  if: ${{ needs.goreleaser.result == 'success' && startsWith(github.ref_name, 'v2.') }}
  uses: MontFerret/.github/.github/workflows/notify-website-release.yml@main
  with:
    tag: ${{ github.ref_name }}
    release-url: ${{ format('https://github.com/{0}/releases/tag/{1}', github.repository, github.ref_name) }}
    commit-sha: ${{ github.sha }}
  secrets:
    FERRET_RELEASE_APP_CLIENT_ID: ${{ secrets.FERRET_RELEASE_APP_CLIENT_ID }}
    FERRET_RELEASE_APP_PRIVATE_KEY: ${{ secrets.FERRET_RELEASE_APP_PRIVATE_KEY }}
```

The caller must make both Ferret Release GitHub App secrets available. The workflow accepts only MontFerret callers, derives the source repository from `github.repository`, and requires the release URL to match that repository and tag. It dispatches the fixed `ferret-release-published` event to `MontFerret/montferret.github.io`; callers cannot change the event type or target repository.

## Publish a validated Ferret Core dependency update

`.github/workflows/publish-ferret-core-update.yml` publishes dependency files that a caller has already prepared, tested, and built. The caller uploads an immutable artifact named `ferret-core-dependency-update` containing exactly root-level `go.mod` and `go.sum` files.

Call the reusable workflow from a job that depends on the preparation job and runs only when the dependency files changed:

```yaml
publish:
  needs: prepare
  if: needs.prepare.outputs.changed == 'true'
  uses: MontFerret/.github/.github/workflows/publish-ferret-core-update.yml@main
  with:
    version: ${{ github.event.client_payload.version }}
    base-sha: ${{ needs.prepare.outputs.base-sha }}
  secrets:
    FERRET_RELEASE_APP_CLIENT_ID: ${{ secrets.FERRET_RELEASE_APP_CLIENT_ID }}
    FERRET_RELEASE_APP_PRIVATE_KEY: ${{ secrets.FERRET_RELEASE_APP_PRIVATE_KEY }}
```

The workflow accepts only CLI, Lab, and Worker callers and revalidates the v2 release tag, base commit, and artifact before authentication. It creates a GitHub App token scoped only to the calling repository, applies only `go.mod` and `go.sum`, and creates or reuses the existing `deps/ferret-core-*` pull request.
