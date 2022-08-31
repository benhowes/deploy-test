# Github Aptible Devops test

This repo contains a very simple deployable image (borrowed from [the aptible demo](https://github.com/aptible/aptible-deploy-action-demo)) and a set of github actions to show a potentially useful way to setup a complete development workflow around this.

Demo environments:

- staging: https://app-44579.on-aptible.com
- production: https://app-44578.on-aptible.com

## How this works

### Environments
This assumes that you have a `staging` and a `production` environment in aptible and appropriate environment secrets set for the corresponding aptible "environment" and "app" set in the environment's secrets.

> **Note**
> Github has an `environment` concept which matches up 1-to-1 with an aptible environment, but it's worth keeping in mind that they are separate but identically named.

### Pull requests
Code is changed in branches and commited as normal.

Pull requests are required to be labled with one of the following labels:
- `version-bump-major`
- `version-bump-minor`
- `version-bump-patch`

This signals the type of change in this PR. e.g. if you're adding a new feature, use the `version-bump-minor` label.

There is an additional CI check which requires that one of the labels is added in order to permit merging to `main`.

> **Note**
> The CI checks are currently empty in this repo since we're just building a demo docker container which does not have any tests/linting we can do.

### On merge to `main`

The CI runs against `main`, and if successful, an action performs the sermver bump. e.g. if your current latest version is `v0.2.0` and you merge a PR with the `version-bump-patch` label, the github action will create a git tag of `v0.2.1` for the latest version.

The action then pushes the tag back to the repo.

### On a new `vX.Y.Z` tag

At this point, a new release has been requested and the actions will:
1. Create a github "release" (e.g. [like this](https://github.com/benhowes/deploy-test/releases/tag/v0.0.6))
2. Build and push a docker container with the latest version of the code
3. Deploy the new version to the `staging` environment


Currently this pushes the docker image to the github container registry as a bot user, then uses the [aptible deploy action](https://github.com/aptible/aptible-deploy-action) to trigger a release in aptible.

> **warning**
> The container registry should be switched to one which supports immutable docker tags so the built image is immune to being overwritten.


### To deploy to `production`
There's a manual workflow ([triggerable here](https://github.com/benhowes/deploy-test/actions/workflows/deploy.yml)) which allows manual deployment of a specified version to any of the available environments.

Github allows review rules to be set on environments and as such, the production environment can be protected.


## Future Work
- [ ] Use an immutable docker registry
- [ ] Verify this meets product requirements
