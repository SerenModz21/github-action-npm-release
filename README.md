# Automatic GitHub Release

> [!NOTE]
> This is a fork of [justincy/github-action-npm-release](https://github.com/justincy/github-action-npm-release) which contains a few changes.
>
> Major changes:
>
> - Updates the action to use Node 24 instead of Node 12
> - Adds apiUrl and messageFormat options (from [@sschoen](https://github.com/sschoen) in <https://github.com/justincy/github-action-npm-release/pull/7>)
> - Resolves the `set-output` deprecation warnings (from [@sschoen](https://github.com/sschoen) in <https://github.com/justincy/github-action-npm-release/pull/7>)
> - Adds `prelease` and `draft` options
> - Create a pre-release if the version includes `beta` or `alpha` and checkVersion is set to true
> - Sets `released` to false when the release is unable to be created

Automatically generate a release when the package.json version changes. The release name and tag will match the new version. If no releases yet exist, this action will create the first release.

The release notes will contain a change log generated from git history in the following format:

```md
- f0d91bd Making progress
- 275e3e2 Initial commit
```

## Assumptions

This action makes a few assumptions:

- `actions/checkout@v6` with `fetch-depth: 0` is used before this action runs. That allows this action to have all the information it needs to generate the change log from the git history.
- You are only releasing from one branch
- It is only used during `push`

## Usage Example

```yml
name: Release
on:
  push:
    branches:
      - master

jobs:
  build:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - name: Release
        uses: SerenModz21/github-action-npm-release@master
        id: release
      - name: Print release output
        if: ${{ steps.release.outputs.released == 'true' }}
        run: echo Release ID ${{ steps.release.outputs.release_id }}
```

Works great in tandem with auto-publishing. Here's an example for the GitHub Package Registry:

```yml
name: Release
on:
  push:
    branches:
      - master

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - name: Automatic GitHub Release
        uses: SerenModz21/github-action-npm-release@master
        id: release
      - uses: actions/setup-node@v6
        if: steps.release.outputs.released == 'true'
        with:
          registry-url: https://npm.pkg.github.com
      - name: Publish
        if: steps.release.outputs.released == 'true'
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

- `token`: Personal access token for GitHub authentication. Optional. Defaults to `${{ github.token }}`.
- `path`: Path of the package.json file that will be examined. Optional. Defaults to `${{ github.workspace }}`.
- `apiUrl`: The URL of the Git(Hub) REST API. Defaults to `${{ github.api_url }}`.
- `messageFormat`: Commit message format (<https://devhints.io/git-log-format>). Defaults to `%h %s`.
- `prelease`: Whether or not to create a pre-release. Defaults to `false`.
- `draft`: Whether or not to create a draft. Defaults to `false`.
- `checkVersion`: Whether or not to create pre-release for beta or alpha versions. Defaults to `true`.

## Outputs

- `released`: Set to true when a release is created.
- `prerelease`: Set to true when the release is a pre-release.
- `draft`: Set to true when the release is a draft.
- `html_url`: The URL for viewing the release in a browser.
- `upload_url`: The URL for uploading assets to the release.
- `release_id`: ID of the release.
- `release_tag`: Tag of the release.
- `release_name`: Name of the release.

## Why re-invent the wheel?

- [Publish to npm](https://github.com/marketplace/actions/publish-to-npm)
- [Release Me Action](https://github.com/ridedott/release-me-action)
  - I don't want to be forced to use any specific commit format. I want a new `version` to be the only signal to release.
- [Version Check](https://github.com/marketplace/actions/version-check)
  - It only checks commits in a single push. If the workflow run associated with the push that has the new version fails and you push again to fix it, then the second workflow run isn't associated with the commits where the version changed and therefore it doesn't detect that the version changed.
- I wanted to learn more about GitHub actions.

## Possible future enhancements

- ~~Add options for draft and pre-release~~ (added by [@sschoen](https://github.com/sschoen))
- ~~Add option for custom git-log format~~ (added by [@sschoen](https://github.com/sschoen))
