<!-- README.md generated by helm-docs from README.md.gotmpl -->
# ziti-host

![Version: 1.1.0](https://img.shields.io/badge/Version-1.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.5.12](https://img.shields.io/badge/AppVersion-1.5.12-informational?style=flat-square)

Reverse proxy cluster services with an OpenZiti tunneler pod

## Requirements

Kubernetes: `>= 1.20.0-0`

## Overview

You may use this chart to publish cluster services to your Ziti network. For example, if you create a Ziti service with a server address of `tcp:kubernetes.default.svc:443` and write a Bind Service Policy assigning the service to the Ziti identity used with this chart, then your Ziti network's authorized clients will be able access this cluster's apiserver. You could do the same thing for any cluster service's domain name.

## How this Chart Works

This chart deploys a pod running `ziti-edge-tunnel`, [the OpenZiti Linux tunneler](https://docs.openziti.io/docs/reference/tunnelers/linux/), in service hosting mode. The chart uses container image `docker.io/openziti/ziti-host` which runs `ziti-edge-tunnel run-host`. This puts the Linux tunneler in "hosting" mode which is useful for binding Ziti services without any need for elevated permissions and without any Ziti nameserver or intercepting proxy. You'll be able to publish any server that is known by an IP address or domain name that is reachable from the pod deployed by this chart.

The enrolled Ziti identity JSON is persisted in a volume, and the chart will migrate the identity from a secret to the volume if the legacy secret exists.

## Installation

```bash
helm repo add openziti https://docs.openziti.io/helm-charts/
```

After adding the charts repo to Helm then you may enroll the identity and install the chart. You may supply a Ziti identity JSON file when you install the chart. This approach enables you to use any option available to the `ziti edge enroll` command.

```bash
ziti edge enroll --jwt /tmp/k8s-tunneler.jwt --out /tmp/k8s-tunneler.json
helm install ziti-host openziti/ziti-host --set-file zitiIdentity=/tmp/k8s-tunneler.json
```

Alternatively, you may supply the JWT directly to the chart. In this case, a private key will be generated on first run and the identity will be enrolled.

```bash
helm install ziti-host openziti/ziti-host --set-file zitiEnrollToken=/tmp/k8s-tunneler.jwt
```

### Installation using an existing secret

**Warning:** this approach does not allow the tunneler to autonomously renew its identity certificate, so you must renew the identity certificate out of band and supply it as an existing secret.

Create the secret:

```bash
kubectl create secret generic k8s-tunneler-identity --from-file=persisted-identity=k8s-tunneler.json
```

Deploy the Helm chart, referring to the existing secret:

```bash
helm install ziti-host openziti/ziti-host --set secret.existingSecretName=k8s-tunneler-identity
```

If desired, change the key name `persisted-identity` with `--set secret.keyName=myKeyName`.

### Identity Directory and Volume

The Ziti identity is stored in a directory inside the container, which is backed by a PersistentVolumeClaim (PVC) by default. This ensures that identity renewals and updates are preserved across pod restarts. If you use an existing secret instead, the identity directory will be read-only, and renewals will not be persisted.

**Warning:** If the identity directory is not writable or not backed by a persistent volume, identity renewals and updates will NOT be preserved across container restarts.

## Values Reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| additionalVolumes | list | `[]` | additional volumes to mount to ziti-host container |
| affinity | object | `{}` |  |
| dnsPolicy | string | `"ClusterFirstWithHostNet"` |  |
| fullnameOverride | string | `""` |  |
| hostNetwork | bool | `false` | bool: host network mode |
| image.args | list | `[]` |  |
| image.pullPolicy | string | `"Always"` |  |
| image.repository | string | `"openziti/ziti-host"` |  |
| imagePullSecrets | list | `[]` |  |
| nameOverride | string | `""` |  |
| nodeSelector | object | `{}` |  |
| podAnnotations | object | `{}` |  |
| podSecurityContext.fsGroup | int | `65534` | int: fsGroup for podSecurityContext (default: nogroup) |
| podSecurityContext.runAsGroup | int | `65534` | int: GID to run the container as (default: nogroup) |
| podSecurityContext.runAsUser | int | `65534` | int: UID to run the container as (default: nobody) |
| ports | list | `[]` |  |
| replicas | int | `1` |  |
| resources | object | `{}` |  |
| secret | object | `{}` |  |
| securityContext | object | `{}` |  |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| spireAgent.enabled | bool | `false` | if you are running a container with the spire-agent binary installed then this will allow you to add the hostpath necessary for connecting to the spire socket |
| spireAgent.spireSocketMnt | string | `"/run/spire/sockets"` | file path of the spire socket mount |
| tolerations | list | `[]` |  |

```bash
helm upgrade {release} {source dir}
```

<!-- README.md generated by helm-docs from README.md.gotmpl -->
