# AdGuard Home Helm Chart

This Helm chart deploys AdGuard Home, a network-wide DNS server with ad-blocking capabilities, on a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure (if persistence is enabled)

## Installing the Chart

To install the chart with the release name `adguard-home`:

```bash
helm install adguard-home ./dns
```

The command deploys AdGuard Home on the Kubernetes cluster with the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `adguard-home` deployment:

```bash
helm delete adguard-home
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

### Global parameters

| Name                      | Description                                     | Value |
| ------------------------- | ----------------------------------------------- | ----- |
| `nameOverride`            | String to partially override adguard-home.name | `""`  |
| `fullnameOverride`        | String to fully override adguard-home.fullname | `""`  |

### Image parameters

| Name                | Description                                          | Value                    |
| ------------------- | ---------------------------------------------------- | ------------------------ |
| `image.repository`  | AdGuard Home image repository                        | `adguard/adguardhome`    |
| `image.tag`         | AdGuard Home image tag (immutable tags are recommended) | `v0.107.65`              |
| `image.pullPolicy`  | AdGuard Home image pull policy                       | `IfNotPresent`           |
| `imagePullSecrets`  | AdGuard Home image pull secrets                      | `[]`                     |

### Deployment parameters

| Name                                    | Description                                                                               | Value   |
| --------------------------------------- | ----------------------------------------------------------------------------------------- | ------- |
| `replicaCount`                          | Number of AdGuard Home replicas to deploy                                                | `2`     |
| `podAnnotations`                        | Annotations for AdGuard Home pods                                                        | `{}`    |
| `podSecurityContext`                    | Set AdGuard Home pod's Security Context                                                  | `{}`    |
| `securityContext`                       | Set AdGuard Home container's Security Context                                            | `{}`    |
| `resources`                             | Set container requests and limits for different resources like CPU or memory             | `{}`    |
| `nodeSelector`                          | Node labels for AdGuard Home pods assignment                                             | `{}`    |
| `tolerations`                           | Tolerations for AdGuard Home pods assignment                                             | `[]`    |
| `affinity`                              | Affinity for AdGuard Home pods assignment (includes pod anti-affinity for node distribution) | `podAntiAffinity configured` |

### Service Account parameters

| Name                         | Description                                                | Value  |
| ---------------------------- | ---------------------------------------------------------- | ------ |
| `serviceAccount.create`      | Specifies whether a ServiceAccount should be created      | `true` |
| `serviceAccount.annotations` | Additional Service Account annotations                     | `{}`   |
| `serviceAccount.name`        | The name of the ServiceAccount to use                     | `""`   |

### Service parameters

| Name                        | Description                                                                   | Value       |
| --------------------------- | ----------------------------------------------------------------------------- | ----------- |
| `service.type`              | AdGuard Home service type                                                     | `ClusterIP` |
| `service.port`              | AdGuard Home web interface service port                                      | `3000`      |
| `service.dnsPort`           | AdGuard Home DNS service port                                                 | `53`        |
| `service.dnsOverTlsPort`    | AdGuard Home DNS-over-TLS service port                                       | `853`       |
| `service.dnsOverHttpsPort`  | AdGuard Home DNS-over-HTTPS service port                                     | `443`       |

### Ingress parameters

| Name                  | Description                                                                                                                      | Value                    |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `ingress.enabled`     | Enable ingress record generation for AdGuard Home                                                                               | `false`                  |
| `ingress.className`   | IngressClass that will be be used to implement the Ingress (Kubernetes 1.18+)                                                  | `""`                     |
| `ingress.annotations` | Additional annotations for the Ingress resource. To enable certificate autogeneration, place here your cert-manager annotations. | `{}`                     |
| `ingress.hosts`       | An array with hosts and paths                                                                                                   | `[{"host": "adguard.local", "paths": [{"path": "/", "pathType": "Prefix"}]}]` |
| `ingress.tls`         | TLS configuration for the Ingress                                                                                               | `[]`                     |

### Persistence parameters

| Name                        | Description                                                | Value              |
| --------------------------- | ---------------------------------------------------------- | ------------------ |
| `persistence.enabled`       | Enable persistence using Persistent Volume Claims         | `true`             |
| `persistence.storageClass`  | Persistent Volume storage class                            | `""`               |
| `persistence.accessMode`    | Persistent Volume access mode                              | `ReadWriteOnce`    |
| `persistence.size`          | Persistent Volume size                                     | `1Gi`              |

### AdGuard Home Configuration parameters

| Name                      | Description                                    | Value        |
| ------------------------- | ---------------------------------------------- | ------------ |
| `config.adminUser`        | Initial admin username                         | `admin`      |
| `config.adminPassword`    | Initial admin password                         | `changeme`   |
| `config.bindHost`         | Web interface bind host                        | `0.0.0.0`    |
| `config.bindPort`         | Web interface bind port                        | `3000`       |
| `config.dnsBindHost`      | DNS server bind host                           | `0.0.0.0`    |
| `config.dnsBindPort`      | DNS server bind port                           | `53`         |

## Configuration and installation details

### Setting up AdGuard Home

After installation, you can access the AdGuard Home web interface to complete the initial setup:

1. If using port-forward: `kubectl port-forward svc/adguard-home 3000:3000`
2. Open your browser and navigate to `http://localhost:3000`
3. Follow the setup wizard to configure your AdGuard Home instance

### Exposing the service

To expose AdGuard Home externally, you have several options:

1. **LoadBalancer service**: Change `service.type` to `LoadBalancer`
2. **NodePort service**: Change `service.type` to `NodePort`
3. **Ingress**: Enable ingress by setting `ingress.enabled=true` and configure the hostname

### High Availability Configuration

The chart is configured with pod anti-affinity by default to ensure that AdGuard Home replicas are scheduled on different nodes for high availability. This is achieved through:

- **Pod Anti-Affinity**: Uses `preferredDuringSchedulingIgnoredDuringExecution` with a weight of 100
- **Topology Key**: `kubernetes.io/hostname` ensures pods are distributed across different nodes
- **Default Replicas**: Set to 2 for redundancy

This configuration provides fault tolerance - if one node fails, AdGuard Home will continue running on the other node(s).

### DNS Configuration

For AdGuard Home to function as your DNS server, you'll need to:

1. Expose the DNS service (port 53) externally
2. Configure your devices or router to use the AdGuard Home IP as the DNS server

## Examples

### Installing with custom values

```bash
helm install adguard-home ./dns \
  --set service.type=LoadBalancer \
  --set persistence.size=5Gi \
  --set config.adminPassword=mySecurePassword
```

### Installing with ingress enabled

```bash
helm install adguard-home ./dns \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=adguard.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
