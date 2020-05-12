# Society

[![Society logo](https://society-assets.s3.ca-central-1.amazonaws.com/social.network+horizontal.png)](https://social.network)

This repository is part of the source code of Society. You can find more information at [social.network](https://social.network) or by contacting support@social.network.

You can find the published source code at [github.com/social-network/society-deployer](https://github.com/social-network/society-deployer).

## Introduction

This repository contains the code and configuration to deploy [society-server](https://github.com/social-network/society-server) and [society-webapp](https://github.com/social-network/society-webapp), as well as dependent components, such as cassandra databases. To allow a maximum of flexibility with respect to where society-server can be deployed (e.g. with cloud providers like AWS, on bare-metal servers, etc), we chose [kubernetes](https://kubernetes.io/) as the target platform.

## Documentation

All the documentation on how to make use of this repository is hosted on https://docs.social.network - refer to the Administrator's Guide.

## Contents

* `ansible/` contains ansible roles and playbooks to install kuberentes, cassandra, etc. See the [Administrator's Guide](https://docs.social.network) for more info.
* `charts/` contains helm charts that can be installed on kubernetes. The charts are mirroed to S3 and can be used with `helm repo add wire https://s3-eu-west-1.amazonaws.com/social-technologies/charts`. See the [Administrator's Guide](https://docs.social.network) for more info.
* `terraform/` contains some examples for provisioning servers. See the [Administrator's Guide](https://docs.social.network) for more info.
* `bin/` contains some helper bash scripts. Some are used in the [Administrator's Guide](https://docs.social.network) when installing wire-server, and some are used for developers/maintainers of this repository.

## For Maintainers of society-server-deployer

### git branches

* `master` branch is the production branch and the one where helm charts are mirrored to S3, and recommended for use. The helm chart mirror can be added as follows: `helm repo add society https://s3-eu-west-1.amazonaws.com/social-technologies/charts`
* `develop` is bleeding-edge, your PRs should branch from here. There is a mirror to S3 you can use if you need to use bleeding edge helm charts: `helm repo add society-develop https://s3-eu-west-1.amazonaws.com/social-technologies/charts-develop`. Note that versioning here is done with git tags, not actual git commits, in order not to pollute the git history.

### developing charts

For local development, instead of `helm install social-technologies/<chart-name>`, use

```bash
./bin/update.sh ./charts/<chart-name> # this will clean and re-package subcharts
helm install charts/<chart-name> # specify a local file path
```

### ./bin/sync.sh

This script is used to mirror the contents of this github repository with S3 to make it easier for us and external people to use helm charts. Usually CI will make use of this automatically on merge to master/develop, but you can also run that manually after bumping versions.
