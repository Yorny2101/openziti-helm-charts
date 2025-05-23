<!-- README.md generated by helm-docs from README.md.gotmpl -->

# zrok

![Version: 1.0.1](https://img.shields.io/badge/Version-1.0.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.4](https://img.shields.io/badge/AppVersion-1.0.4-informational?style=flat-square)

Run the zrok controller and zrok frontend components as a K8s deployment

## Overview

## Requirements

### Add the OpenZiti Charts Repo to Helm

```bash
helm repo add openziti https://docs.openziti.io/helm-charts/
```

## Minimal Example with Nginx Ingress

This example does not configure TLS termination for the API or public shares, metrics, or limits. You must configure a
wildcard DNS record (A record) that resolve to the value of `ZROK_DNS_ZONE`.

Use an `sslip.io` wildcard/zone like `zrok.192.168.49.2.sslip.io` for testing and tiny scale deployments if you
want to avoid setting up DNS. This works with any IP address.

```bash
ZROK_DNS_ZONE=zrok.example.com
ZITI_NAMESPACE=miniziti
ZITI_MGMT_API_HOST=ziti-controller-client.${ZITI_NAMESPACE}.svc.cluster.local
ZITI_PWD=$(kubectl -n "${ZITI_NAMESPACE}" get secrets "ziti-controller-admin-secret" \
    --output go-template='{{index .data "admin-password" | base64decode}}')

helm upgrade \
    --install \
    --namespace zrok --create-namespace \
    --values https://openziti.io/helm-charts/charts/zrok/values-ingress-nginx.yaml \
    --set "ziti.advertisedHost=${ZITI_MGMT_API_HOST}" \
    --set "ziti.password=${ZITI_PWD}" \
    --set "dnsZone=${ZROK_DNS_ZONE}" \
    --set "controller.ingress.hosts[0]=api.${ZROK_DNS_ZONE}" \
    zrok openziti/zrok
```

## TLS termination with Nginx

One way to terminate TLS with Nginx is to use Cert Manager. Cert Manager will issue a certificate, store it in the specified Secret, and configure the Ingress to use the certificate. This example shows the default behavior to use the Ingress host(s) as DNS SANs.

1. Install Cert Manager
1. Create a ClusterIssuer with a Let's Encrypt account and DNS challenge solver. Solving the DNS challenge is one way
    for Cert Manager to obtain a wildcard certificate which is necessary for zrok frontend's Ingress.
1. Set input values to annotate zrok's Ingresses with the name of the ClusterIssuer and specify a TLS secret name.

    ```bash
    helm upgrade zrok \
        --set "controller.ingress.annotations=cert-manager.io/cluster-issuer: letsencrypt-prod" \
        --set "controller.ingress.tlsSecretName=zrok-api-tls" \
        --set "frontend.ingress.annotations=cert-manager.io/cluster-issuer: letsencrypt-prod" \
        --set "frontend.ingress.tlsSecretName=zrok-wildcard-tls" \
        openziti/zrok
    ```

## Default account

The chart automatically creates a zrok account in the database. You can use the account token to enable a device environment with `zrok enable ${ZROK_ENABLE_TOKEN}` and you can log in to the zrok console with the username and password.

Get the zrok account token:

```bash
kubectl -n zrok \
    get secrets zrok-ziggy-account-token \
    -o go-template='{{"\n"}}{{index .data "token" | base64decode }}{{"\n"}}'
```

```text title="Output"

qEP0MNtA74T3

```

Get the zrok console login credentials:

```bash
kubectl -n zrok \
    get secrets zrok-ziggy-account-password \
    -o go-template='{{"\n"}}{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```

```text title="Output"

password: p7XWVyjHbMWazLc6PZveF2b8SB2wzxDD
username: ziggy@zrok.192.168.49.2.sslip.io

```

The zrok console URL depends on how you configure ingress. If you used the NGINX Ingress example, then you can query the URL with:

```bash
kubectl -n zrok get ingress zrok 
```

```text title="Output"
NAME   CLASS   HOSTS                             ADDRESS        PORTS   AGE
zrok   nginx   api.zrok.192.168.49.2.sslip.io    192.168.49.2   80      8m41s
```

## Values Reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| autoscaling.enabled | bool | `false` |  |
| autoscaling.maxReplicas | int | `100` |  |
| autoscaling.minReplicas | int | `1` |  |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| controller.email | object | `{}` | send invitation acknowledgements and usage limit warnings from the specified email address |
| controller.extraConfig | object | `{}` | append additional controller config |
| controller.ingress.annotations | object | `{}` | The annotations to use for the zrok controller ingress resource |
| controller.ingress.className | string | `""` | The ingress class to use for the zrok controller |
| controller.ingress.enabled | bool | `false` | enable the ingress resource for  |
| controller.ingress.hosts | list | `[]` | domain names for the zrok controller ingress resource; first is used for email templates |
| controller.ingress.scheme | string | `"https"` | URI scheme to advertise for the controller's ingress resource |
| controller.ingress.tls | list | `[]` | ingress tls configurations for the zrok controller ingress resource |
| controller.invites.open | bool | `true` | enable the zrok controller to onboard new users when they run "zrok invite" |
| controller.invites.token_required | bool | `false` | require new users to submit an invitation token when they run "zrok invite", tokens are generated with "zrok admin generate" |
| controller.metrics.agent.source.type | string | `"websocketSource"` | initiate a WebSocket connection to the Ziti Management API URL to receive fabric usage metrics |
| controller.metrics.enabled | bool | `false` | enable metrics collection and reporting for the zrok controller |
| controller.metrics.limits.bandwidth.per_account.limit.rx | int | `-1` | per-account limit threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.limit.total | int | `10485760` | per-account limit threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.limit.tx | int | `-1` | per-account limit threshold for transmit bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.period | string | `"5m"` | per-account period for bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.warning.rx | int | `-1` | per-account warning threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.warning.total | int | `7242880` | per-account warning threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_account.warning.tx | int | `-1` | per-account warning threshold for transmit bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.limit.rx | int | `-1` | per-environment limit threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.limit.total | int | `-1` | per-environment limit threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.limit.tx | int | `-1` | per-environment limit threshold for transmit bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.period | string | `"5m"` | per-environment period for bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.warning.rx | int | `-1` | per-environment warning threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.warning.total | int | `-1` | per-environment warning threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_environment.warning.tx | int | `-1` | per-environment warning threshold for transmit bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.limit.rx | int | `-1` | per-share limit threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.limit.total | int | `-1` | per-share limit threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.limit.tx | int | `-1` | per-share limit threshold for transmit bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.period | string | `"5m"` | per-share period for bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.warning.rx | int | `-1` | per-share warning threshold for receive bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.warning.total | int | `-1` | per-share warning threshold for total bandwidth usage |
| controller.metrics.limits.bandwidth.per_share.warning.tx | int | `-1` | per-share warning threshold for transmit bandwidth usage |
| controller.metrics.limits.cycle | string | `"5m"` | the period for checking usage limits |
| controller.metrics.limits.enforcing | bool | `false` | toggle enforcement of usage limits |
| controller.metrics.limits.environments | int | `-1` | the maximum number of environments |
| controller.metrics.limits.shares | int | `-1` | the maximum number of shares |
| controller.persistence.VolumeName | string | `nil` | PVC volume name |
| controller.persistence.accessMode | string | `"ReadWriteOnce"` | PVC access mode: ReadWriteOnce (concurrent mounts not allowed), ReadWriteMany (concurrent allowed) |
| controller.persistence.annotations | object | `{}` |  |
| controller.persistence.enabled | bool | `true` | storage claim for the zrok controller database if using sqlite3 |
| controller.persistence.existingClaim | string | `""` | If defined, PVC must be created manually before volume will be bound |
| controller.persistence.mount_dir | string | `"/var/lib/zrok"` | The mount path for the zrok controller database |
| controller.persistence.size | string | `"2Gi"` | PVC size of zrok controller database |
| controller.persistence.storageClass | string | `""` | Storage class of PV to bind. By default it looks for the default storage class. If the PV uses a different storage class, specify that here. |
| controller.service.advertisedPort | int | `80` | The port to advertise for the zrok controller service |
| controller.service.containerPort | int | `18080` | The port to expose on the zrok controller container |
| controller.service.type | string | `"ClusterIP"` | The service type to use for the zrok controller |
| controller.specVersion | int | `4` |  |
| dnsZone | string | `"zrok.example.com"` | The DNS zone with a wildcard * A record to use for the zrok public frontend |
| frontend.extraConfig | object | `{}` | append additional frontend config |
| frontend.homeDir | string | `"/var/lib/zrok"` | a read-only mountpoint for the frontend's Ziti identity is "homeDir" because zrok always looks in $HOME/.zrok/identities |
| frontend.ingress.annotations | object | `{}` | The annotations to use for the frontend's ingress resource |
| frontend.ingress.className | string | `""` | The annotations to use for the frontend's ingress resource |
| frontend.ingress.enabled | bool | `false` | enable the frontend's ingress resource |
| frontend.ingress.hosts | list | `[]` | *.{{ .Values.dnsZone }} is always set when ingress enabled; specify optional, additional wildcard hostnames to use for the frontend's ingress resource |
| frontend.ingress.scheme | string | `"https"` | URI scheme to advertise for the frontend's ingress resource |
| frontend.ingress.tls | list | `[]` | The TLS configuration for the frontend's ingress resource |
| frontend.service.advertisedPort | int | `80` | The port to advertise for the zrok frontend service |
| frontend.service.containerPort | int | `8080` | The port to expose on the zrok frontend container |
| frontend.service.type | string | `"ClusterIP"` | The service type to use for the zrok frontend |
| frontend.specVersion | int | `3` |  |
| fullnameOverride | string | `""` |  |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"openziti/zrok"` |  |
| image.tag | string | `""` | Overrides the image tag whose default is the chart appVersion. |
| imagePullSecrets | list | `[]` |  |
| influxdb2.adminUser.existingSecret | string | `""` | The name of an existing secret with admin-password and admin-token for the InfluxDB service |
| influxdb2.adminUser.password | string | `"admin"` | The admin password for the InfluxDB service |
| influxdb2.adminUser.username | string | `"admin"` | The admin username for the InfluxDB service |
| influxdb2.enabled | bool | `false` | enable the influxdb2 subchart |
| influxdb2.service.port | int | `8086` | The port to advertise for the InfluxDB service |
| influxdb2.service.type | string | `"ClusterIP"` | The service type to use for the InfluxDB service |
| influxdb2.service.url | string | `""` | set URL of the InfluxDB service if subchart is disabled |
| nameOverride | string | `""` |  |
| nodeSelector | object | `{}` |  |
| podAnnotations | object | `{}` |  |
| podSecurityContext | object | `{"fsGroup":2171,"runAsGroup":2171,"runAsUser":2171}` | default pod security context for all containers |
| podSecurityContext.fsGroup | int | `2171` | volume mount filesystem group owner |
| podSecurityContext.runAsGroup | int | `2171` | effective GID |
| podSecurityContext.runAsUser | int | `2171` | effective UID |
| replicaCount | int | `1` |  |
| resources | object | `{}` |  |
| securityContext | object | `{}` | override the default pod security context for init and app containers |
| serviceAccount.annotations | object | `{}` | Annotations to add to the service account |
| serviceAccount.name | string | `""` | The name of the service account to use. If not set and create is true, a name is generated using the fullname template |
| test.backoffLimit | int | `3` | retry until first success unless backoffLimit is reached |
| test.enabled | bool | `false` | run the 'zrok test loopback public' in a one-off Job to verify the zrok public frontend is working |
| tolerations | list | `[]` |  |
| ziti.advertisedHost | string | `"localhost"` | The Ziti Management API host to bootstrap with zrok and to collect fabric metrics from |
| ziti.advertisedPort | string | `"443"` | The Ziti Management API port |
| ziti.ca_cert_configmap | string | `"ziti-controller-ctrl-plane-cas"` | name of the configmap containing the Ziti CA certificate trust bundle that trust-manager syncs to namespaces with the label "openziti.io/namespace: enabled"; has format {{controller's Helm release name}}-ctrl-plane-cas |
| ziti.ca_cert_dir | string | `"/etc/ziti"` | mountpoint of the Ziti CA certificate trust bundle |
| ziti.ca_cert_file | string | `"ctrl-plane-cas.crt"` | key name of trust bundle in configmap and filename to project into mountpoint |
| ziti.password | string | `"admin"` | Ziti admin login password |
| ziti.username | string | `"admin"` | Ziti admin login name |

<!-- README.md generated by helm-docs from README.md.gotmpl -->
