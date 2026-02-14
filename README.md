# Typesense Helm Chart

This repository contains a Helm chart for deploying the Typesense search engine.

## Overview

Typesense is a fast, typo-tolerant search engine for building delightful search experiences. This Helm chart simplifies the deployment of Typesense on Kubernetes.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+

## Package

After making changes to the Chart itself you need to update the version number in `Chart.yaml` an package the Chart

```sh
helm package charts/typesense
helm repo index --url https://newtrition-data.github.io/helm-charts/ .
```

## Installation

To install the chart with the release name `my-release`:

```sh
helm repo add newtrition-data https://newtrition-data.github.io/helm-charts
helm install my-release newtrition-data/typesense
```

## Uninstallation

To uninstall/delete the `my-release` deployment:

```sh
helm uninstall my-release
```

## Configuration

The following table lists the configurable parameters of the Typesense chart and their default values.

| Parameter                        | Description                                                  | Default                        |
|----------------------------------|--------------------------------------------------------------|--------------------------------|
| `replicaCount`                   | Number of replicas                                           | `1`                            |
| `image.repository`               | Image repository                                             | `typesense/typesense`          |
| `image.pullPolicy`               | Image pull policy                                            | `IfNotPresent`                 |
| `image.tag`                      | Image tag                                                    | `29.0`                         |
| `imagePullSecrets`               | Secrets for image pull                                       | `[]`                           |
| `nameOverride`                   | Override the chart name                                      | `""`                           |
| `fullnameOverride`               | Override the full name                                       | `""`                           |
| `persistence.enabled`            | Enable persistence                                           | `false`                        |
| `persistence.storageClass`       | Storage class for persistence                                | `""`                           |
| `persistence.size`               | Size of persistent storage                                   | `5Gi`                          |
| `persistence.accessModes`        | Access modes for persistent storage                          | `["ReadWriteOnce"]`            |
| `apiKey`                         | API key for Typesense                                        | `""`                           |
| `cors`                           | Enable CORS                                                  | `false`                        |
| `service.type`                   | Service type                                                 | `ClusterIP`                    |
| `service.port`                   | Service port                                                 | `8108`                         |
| `resources.requests.cpu`         | CPU resource requests                                        | `100m`                         |
| `resources.requests.memory`      | Memory resource requests                                     | `128Mi`                        |
| `resources.limits.cpu`           | CPU resource limits                                          | `250m`                         |
| `resources.limits.memory`        | Memory resource limits                                       | `500Mi`                        |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```sh
helm install my-release newtrition-data/typesense --set replicaCount=3
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example:

```sh
helm install my-release newtrition-data/typesense -f values.yaml
```

## Accessing the Chart Repository

The Helm chart repository can be accessed at the following URL: [https://newtrition-data.github.io/helm-charts/index.yaml](https://newtrition-data.github.io/helm-charts/index.yaml)

## License

This project is licensed under the MIT License. See the LICENSE file for details.
