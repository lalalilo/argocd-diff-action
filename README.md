# ArgoCD Diff GitHub Action

This action generates a diff between the current PR and the current state of the cluster.

Note that this includes any changes between your branch and latest `main`, as well as ways in which the cluster is out of sync.

This is a fork of the original [argocd-diff-action](https://github.com/quizlet/argocd-diff-action) repository. This fork was created to update the github action in order for it to work with more recent versions of ArgoCD by removing the API call to get the list of ArgoCD apps and using the CLI instead.

## Usage

Example GH action:
```yaml
name: ArgoCD Diff

on:
  pull_request:
    branches: [main]

jobs:
  argocd-diff:
    name: Generate ArgoCD Diff
    runs-on: ubuntu-latest
    steps:

      - name: Checkout repo
        uses: actions/checkout@v2

      - uses: lalalilo/argocd-diff-action@main
        name: ArgoCD Diff
        with:
          argocd-server-url: argocd.example.com
          argocd-token: ${{ secrets.ARGOCD_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          argocd-version: v1.6.1
          argocd-extra-cli-args: --grpc-web
```

## How it works

1. Downloads the specified version of the ArgoCD binary, and makes it executable
2. Uses the binary to run `argocd app list` to get all the apps associated with the current repository
3. Filters the apps to only keep the ones synced to `main` branch
4. Runs `argocd app diff` for each app
5. Posts the diff output as a comment on the PR

## Publishing

Build the script and commit to your branch:
`yarn run build && yarn run pack`
Commit the build output, and make a PR.

## Issues

This works for ArgoCD version up to 2.6.x, with ArgoCD version 2.7 this will no longer work as by default the `argocd app diff` will be done server side (for now it is not the default, it is only done if you pass the flag `--server-side-generate`). The server side diffing does not work for now, there is an ongoing issue to fix it: https://github.com/argoproj/argo-cd/issues/8145
