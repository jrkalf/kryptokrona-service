# Payment gateway docker image for Kryptokrona
When running a Kryptokrona (XKR) mining pool, you'll need a payment gateway / wallet service which handles the payout towards miners. This repository provides the means to build such a gateway using a docker image. The original source, to be found [here](https://github.com/kryptokrona/kryptokrona), is used during the build of the image. The deployment files in this repository help you host the docker image in a kubernetes environment.

# What is Kryptokrona?
[Kryptokrona](https://kryptokrona.org) is a decentralized cryptocurrency founded in the Scandinavian region of Europe. Sending and receiving money should not be expensive or slow. Kryptokrona works with open source code that allows you to be involved and improve the communication and money of the future. The source can be found on [Github](https://github.com/kryptokrona/kryptokrona).

## Quick reference
- **Maintained by**: [Jelle Kalf](https://github.com/jrkalf)
- **Supported architectures**: `arm64v8`, `amd64`
- **Supported tags**: `latest`, `v1.1.5`, `v1.1.3`, ~`v1.1.2`~, ~`v1.1.1`~

*Do not use older versions unless explicitely asked by the maintainers. Version 1.1.3 is the latest as of December 2023!*

# What is the difference to Kryptokrona's Docker build?
The Dockerfile here is available for learning and optimisation. Pre-built Docker images are available [here](https://hub.docker.com/repository/docker/jrkalf/kryptokrona-service/). These Dockerfiles are trimmed down to the bare minimum of running the Payment Gateway Service only. 

# How to use this image
There are two ways of consuming the images:

0. Prerequisits
1. Kubernetes
2. Docker Compose (being built)

## Prerequisits
The Kryptokrona service required connectivity with a blockchain node. You can pick a publicly available node for this, or setup your own node using the [Kryptokrona Blockchain Node](https://github.com/jrkalf/kryptokrona-node/) also available. The Payment Gateway Service requires you to have port 8070 available to the outside world.

## Kubernetes

**Step 1:** Clone the github repo:

```console
$ git clone https://github.com/jrkalf/kryptokrona-service.git && cd kryptokrona-service
```

In this directory you'll find 4 files you need to modify to run the Payment gateway:
- services.yaml
- storage.yaml
- secrets.yaml
- deployment.yaml


**Step 2:** Create a *namespace* for our Kryptokrona application (optional but recommended):

```console
$ kubectl create ns kryptokrona
```

*If you already run the [kryptokrona-xmrig miner](https://github.com/jrkalf/kryptokrona-xmrig/) or [kryptokrona-node](https://github.com/jrkalf/kryptokrona-node/) in your kubernetes, chances are high you already have a namespace called kryptokrona.*

**Step 3:** Edit the [`deployment.yaml`](https://github.com/jrkalf/kryptokronan-service/blob/main/deployment.yaml) file. Things you may want to modify include:
- `replicas`: number of desired pods to be running. As I run a 3 worker node Turing Pi cluster, I run 3 replica's
- `image:tag`: to view all available versions, go to the [Tags](https://hub.docker.com/repository/docker/jrkalf/xmrig-kryptokrona/tags) tab of the Docker Hub repo.
- `NODE_ARGS`: These are cli options that will be passed to the kryptokronad at runtime.
- `resources`: set appropriate values for `cpu` and `memory` requests/limits.
- `affinity`: the manifest will schedule only one pod per node, if that's not the desired behavior, remove the `affinity` block.

**Step 4:** Edit the [`services.yaml`](https://github.com/jrkalf/kryptokronan-service/blob/main/services.yaml) file. Things you may want to modify include:
- `namespace`: This must match the namespace from **step 2**.
- `type`: This type must match the type of exposure you're running. Whether it's a clusterIP or LoadBalancer.

**Step 5:** Edit the [`storage.yaml`](https://github.com/jrkalf/kryptokronan-service/blob/main/storage.yaml) file. Things you may want to modify include:
- `name`: Make sure this matches the `volumeName` in the persistentvolumeclaim section.
- `storageClassName`: This must match with the name of your local storage provider. **example: I use nfs-client as name for my nfs-external-subdir-provisioner**
- `nfs`: Adjust your settings accordingly. Hint is in the file.

**Step 6:** Edit the ['secrets.yaml'](https://github.com/jrkalf/kryptokronan-service/blob/main/secrets.yaml) file.
This file holds the password to the wallet file! Do not expose this to the internet, do not store this in the pod. Keep it safe and use a kubernetes secret.

**Step 7:** Once you are satisfied with the above manifests, create a *deployment*:

```console
$ kubectl apply -f services.yaml \
    -f persistentvolume.yaml \
    -f persistentvolumeclaim.yaml \
    -f secrets.yaml
    -f deployment.yaml
```
## Logging

For Kubernetes run:
```console
$ kubectl logs --follow -n kryptokrona <pod-name> 
```
## Disclaimer
Use at your own discression. This repository is by no means financial advise to mine cryptocurrency. 
This is a project to learn how to build containerised applications.

## License
The Docker image is licensed under the terms of the [MIT License](https://github.com/jrkalf/kryptokrona-service/blob/main/LICENSE). XMRig is licensed under the GNU General Public License v3.0. See its [`LICENSE`](https://github.com/xmrig/xmrig/blob/master/LICENSE) file for details.

## Used works from other repositories
This repo is a based on works of:
- [Kryptokrona](https://kryptokrona.org) / [Kryptokrona Github](https://github.com/kryptokrona/kryptokrona)

## Contact 
- Find me on Kryptokrona's [Hugin Messenger](https://hugin.chat) at addres `SEKReT7odTKSJjXs9BqKAwAEZhW8XuiowdFd4MjTUidc11fur3TpGjPeKKqaCzZdbF3YBf1RfqFowD8WrWAei5grQXb8fXujX7K`.
- Find me on [github](https://github.com/jrkalf/), [Mastodon](https://mastodon.nl/@jelle77) or [Twitter](https://twitter.com/jkalf).
