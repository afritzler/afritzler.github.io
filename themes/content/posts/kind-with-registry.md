+++
categories = ["kubernetes", "development", "cloud"]
date = "2020-03-02"
description = "Using Kind together with a Local Docker Registry"
featuredalt = ""
featuredpath = "date"
linktitle = ""
slug = "Using Kind together with a Local Docker Registry"
title = "Using Kind together with a Local Docker Registry"
type = ["posts","post"]
comments = "true"
tags = ["kubernetes", "development", "cloud"]
+++

If you want to test your Docker images inside a Kubernetes cluster before pushing them to a public repository, using `kind` in conjunction with a local repository might be just the solution for you.

Just create a script call `kind-with-registry.sh` which contains the following code

```shell script
#!/bin/sh
set -o errexit

# desired cluster name; default is "kind"
KIND_CLUSTER_NAME="${KIND_CLUSTER_NAME:-kind}"

# create registry container unless it already exists
reg_name='kind-registry'
reg_port='5000'
running="$(docker inspect -f '{{.State.Running}}' "${reg_name}" 2>/dev/null || true)"
if [ "${running}" != 'true' ]; then
  docker run \
    -d --restart=always -p "${reg_port}:5000" --name "${reg_name}" \
    registry:2
fi
reg_ip="$(docker inspect -f '{{.NetworkSettings.IPAddress}}' "${reg_name}")"

# create a cluster with the local registry enabled in containerd
cat <<EOF | kind create cluster --name "${KIND_CLUSTER_NAME}" --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:${reg_port}"]
    endpoint = ["http://${reg_ip}:${reg_port}"]
EOF
```

This will spin up a `kind` cluster plus a local registry which is located under `localhost:5000`. `docker ps` should produce the following output if everything went well

```shell script
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
7b88dbd4757b        kindest/node:v1.17.0   "/usr/local/bin/entr…"   11 minutes ago      Up 11 minutes       127.0.0.1:32769->6443/tcp   kind-control-plane
e40321acce8b        registry:2             "/entrypoint.sh /etc…"   14 minutes ago      Up 14 minutes       0.0.0.0:5000->5000/tcp      kind-registry
```

## Usage

To use your local setup run the following command

```shell script
# inside your Dockerfile folder
docker build localhost:5000/hello-app:1.0 .
docker push localhost:5000/hello-app:1.0
kubectl create deployment hello-server --image=localhost:5000/hello-app:1.0
```

## References

This post is based on the official `kind` [documentation](https://kind.sigs.k8s.io/docs/user/local-registry/). Additional information can be also found in the [kind](https://github.com/kubernetes-sigs/kind) GitHub repository.
