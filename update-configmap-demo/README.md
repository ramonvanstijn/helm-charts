# update-configmap-demo

This demo shows how an update to the configuration file default.conf from nginx (which it only reads at startup) triggers a rolling update.

## Problem

Applications that read a configuration file when they start up, need to be restarted after the configuration file changed. How does this work when I use Helm to deploy my application to Kubernetes?

## Solution

As described in the Helm documentation, an annotation with the checksum of the ConfigMap needs to be added to the Deployment:

```yaml
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
[...]
```

The same source also states:

See also the `helm upgrade --recreate-pods` flag for a slightly different way of addressing this issue.

I tested that way and it always restarted the pods which isn't the desired behaviour.

The third way I discovered is to put the checksum of the configuration file in an environment variable, for the full example see https://boxboat.com/2018/07/05/gitops-kubernetes-rolling-update-configmap-secret-change/

## Demo scenario

1. Clone this repository
2. Install the chart
3. Open the URL in the browser
4. Change the line `return 200 'Hello, world!';` in `files/default.conf` to something else, for example `return 200 'Hello? Is it me you are looking for?';`
5. Upgrade the release
   >You should already see the pods restarting
6. Refresh the page

## Configuration

Usually the demo works well with the default values.yaml.

| Parameter                          | Description                                                               | Default           |
| ---------------------------------- | ------------------------------------------------------------------------- | ------------------|
| `replicaCount`                     | Number of replicas                                                        | `2`               |
| `image.repository`                 | Container image repository                                                | `nginx`           |
| `image.tag`                        | Container image tag                                                       | `alpine`          |
| `image.pullPolicy`                 | Container pull policy                                                     | `IfNotPresent`    |
| `service.type`                     | Service type                                                              | `NodePort`        |
| `service.port`                     | Service port                                                              | `80`              |
| `ingress.enabled`                  | Ingress enabled                                                           | `false`           |