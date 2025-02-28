ESO and the ESO Helm Chart have two distinct lifecycles and can be released independently. Helm Chart releases are named `external-secrets-x.y.z`.

The external-secrets project is released on a as-needed basis. Feel free to open a issue to request a release.

## Release ESO

1. Run `Create Release` Action to create a new release, pass in the desired version number to release.
    1. note: choose the right `branch` to execute the action: use `main` when creating a new release. Use `release-x.y` when you want to bump a LTS release.
1. GitHub Release, Changelog will be created by the `release.yml` workflow which also promotes the container image.
1. update Helm Chart, see below
1. update OLM bundle, see [helm-operator docs](https://github.com/external-secrets/external-secrets-helm-operator/blob/main/docs/release.md#operatorhubio)

## Release Helm Chart

1. Update `version` and/or `appVersion` in `Chart.yaml` and run `make helm.docs helm.update.appversion`
1. If there is any CRD change, run `make helm.test.update` and `make helm.test`
1. push to branch and open pr
1. run `/ok-to-test-managed` commands for all cloud providers
1. merge PR if everyhing is green
1. CI picks up the new chart version and creates a new GitHub Release for it
1. create/merge into release branch
    1. on a `minor` release: create a new branch `release-x.y`
    1. on a `patch` release: merge main into `release-x.y`

## Release OLM Bundle

In order to make the latest release available to [OperatorHub.io](https://operatorhub.io/) we need to create a bundle and open a PR in the [community-operators](https://github.com/k8s-operatorhub/community-operators/) repository.

To create a bundle first increment the `VERSION` in the Makefile as described above. Then run the following commands in the root of the repository:

```bash
# clone repo
git clone https://github.com/external-secrets/external-secrets-helm-operator
cd external-secrets-helm-operator

# bump version
export VERSION=x.y.z
sed -i "s/^VERSION ?= .*/VERSION ?= ${VERSION}/" Makefile

# prep release
make prepare-stable-release
```

Check the generated files in the `bundle/` directory. If they look good add & commit them, open a PR against this repository. You can always use the [OperatorHub.io/preview](https://operatorhub.io/preview) page to preview the generated CSV.

```bash
git status
git add .
git commit -s -m "chore: bump version xyz"
git push
```

Once the PR is merged we need create a pull request against both community-operators repositories. There's a make target that does the heavy lifting for you:
```bash
make bundle-operatorhub
```

This command will add/commit/push and open pull requests in the respective repositories.