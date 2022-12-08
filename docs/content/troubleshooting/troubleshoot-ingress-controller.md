---
title: "Troubleshooting Common Issues"
date: 2021-07-13T21:01:29-06:00
draft: true
description: "This document describes how to troubleshoot problems with the Ingress Controller."
# Assign weights in increments of 100
weight: 100
draft: false
toc: true
tags: [ "docs" ]
# Taxonomies
# These are pre-populated with all available terms for your convenience.
# Remove all terms that do not apply.
doctypes: ["troubleshooting"]
docs: "DOCS-619"
---

## Overview

This document describes how to troubleshoot problems with the Ingress Controller.

## Potential Problems

The table below categorizes some potential problems with the Ingress Controller you may encounter and suggests how to troubleshoot those problems using one or more methods from the next section.

{{% table %}}
| Problem Area | Symptom | Troubleshooting Method | Common Cause |
|-----|-----|-----|-----|
| Start | The Ingress Controller fails to start. | Check the logs. | Misconfigured RBAC, a missing default server TLS Secret.|
| Ingress Resource and Annotations | The configuration is not applied | Check the events of the Ingress resource, check the logs, check the generated config. | Invalid values of annotations. |
| VirtualServer and VirtualServerRoute Resources | The configuration is not applied. | Check the events of the VirtualServer and VirtualServerRoutes, check the logs, check the generated config. | VirtualServer or VirtualServerRoute is invalid. |
| Policy Resource | The configuration is not applied. | Check the events of the Policy resource as well as the events of the VirtualServers that reference that policy, check the logs, check the generated config. | Policy is invalid. |
| ConfigMap Keys | The configuration is not applied. | Check the events of the ConfigMap, check the logs, check the generated config.  | Invalid values of ConfigMap keys. |
| NGINX | NGINX responds with unexpected responses. | Check the logs, check the generated config, check the live activity dashboard (NGINX Plus only), run NGINX in the debug mode. | Unhealthy backend pods, a misconfigured backend service. |
{{% /table %}}

## Troubleshooting Methods

Note that the commands in the next sections make the following assumptions:
* The Ingress Controller is deployed in the namespace `nginx-ingress`.
* `<nginx-ingress-pod>` is the name of one of the Ingress Controller pods.

### Checking the Ingress Controller Logs

To check the Ingress Controller logs -- both of the Ingress Controller software and the NGINX access and error logs -- run:
```
$ kubectl logs <nginx-ingress-pod> -n nginx-ingress
```

### Enabling debuging for NGINX Ingress Controller

If you need to do additional troubleshooting for NGINX Ingress controller, there are few additional settings you can configure, to add more verbose logging.

There are two settings that need to be set to enable more debug/verbose logging for NGINX Ingress controller.

1. command line arguments
2. configmap settings

This document will cover how to enable both.


You can use NGINX Ingress controller command line arguments. This is a great way to increase debug log levels for both Ingress as well as NGINX.
When using `manifest` for deployment:

1.) Use the command line argument, `- -nginx-debug` in your deployment/daemonset
2.) If you want to increase the Ingress `Controller` process lets, set the command line argument to `v=3`, which collects more information on Ingress

 ```yaml
 -v=3
 ```

Here is a small snippet of setting these a command line arguments in the `ARGS` section of a deployment:

```yaml
args:
  - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
  - -enable-cert-manager
  - -nginx-debug
  - -v=3
```

You can configure `error-log-level` in the NGINX Ingress controller `configMap`:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
  namespace: nginx-ingress
data:
  error-log-level: "debug"
 ```

## When using `Helm`

If you are using `helm`, you can look for these two settings:
```
controller.nginxDebug = true or false
controller.loglevel = 1 to 3 value
```
For example, if using a `values.yaml` file:

```yaml
  ## Enables debugging for NGINX. Uses the nginx-debug binary. Requires error-log-level: debug in the ConfigMap via `controller.config.entries`.
  nginxDebug: true

  ## The log level of the Ingress Controller.
  logLevel: 3
```
Here is a more complete `values.yaml` file when using `helm`:

```yaml
controller:
  kind: Deployment
  nginxDebug: true
  logLevel: 3
  annotations:
    nginx: ingress-prod
  pod:
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "9113"
      prometheus.io/scheme: http
    extraLabels:
      env: prod-weset
  nginxplus: plus
  image:
    repository: nginx/nginx-ingress
    tag: 2.4.1
  # NGINX Configmap
  config:
    entries:
      error-log-level: "debug"
      proxy_connet_timeout: "5s"
      http-snippets: |
        underscores_in_headers on;
  ingressClass: nginx
```

By enabling the `nginx-debug` CLI argument and changing the `error-log-level` to `debug`, you can capture more output and debug any issues that are going on.
**NOTE**: It is recommended to only use the `nginx-debug` CLI and the `error-log-level` enabled when debugging purposes.

## Example of debugging output for NGINX Ingress controller pod.

Once you have debugging enabled, you can see the logs have additional entries when reviewing logs:

```nginx
I1026 15:39:03.269092       1 manager.go:301] Reloading nginx with configVersion: 1
I1026 15:39:03.269115       1 utils.go:17] executing /usr/sbin/nginx-debug -s reload -e stderr
2022/10/26 15:39:03 [notice] 19#19: signal 1 (SIGHUP) received from 42, reconfiguring
2022/10/26 15:39:03 [debug] 19#19: wake up, sigio 0
2022/10/26 15:39:03 [notice] 19#19: reconfiguring
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF0A420:16384 @16
2022/10/26 15:39:03 [debug] 19#19: add cleanup: 000056362AF0C318
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF48230:16384 @16
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF00DE0:4096
2022/10/26 15:39:03 [debug] 19#19: read: 46, 000056362AF00DE0, 3090, 0
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF58670:16384 @16
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF12440:4280
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF13500:4280
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF145C0:4280
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF5C680:4280
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF5D740:4280
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF5E800:4280
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF5F8C0:16384 @16
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF41500:4096
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF638D0:8192
2022/10/26 15:39:03 [debug] 19#19: include /etc/nginx/mime.types
2022/10/26 15:39:03 [debug] 19#19: include /etc/nginx/mime.types
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF658E0:4096
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF668F0:5349
2022/10/26 15:39:03 [debug] 19#19: read: 47, 000056362AF658E0, 4096, 0
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF67DE0:4096
2022/10/26 15:39:03 [debug] 19#19: read: 47, 000056362AF658E1, 1253, 4096
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF68DF0:16384 @16
2022/10/26 15:39:03 [debug] 19#19: posix_memalign: 000056362AF6CE00:16384 @16
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AF70E10:524288
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362AFF0E20:524288
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362B070E30:524288
2022/10/26 15:39:03 [debug] 19#19: malloc: 000056362B0F0E40:400280
```

Once you have completed your debugging process, you can change the values back to the original values.




### Checking the Live Activity Monitoring Dashboard

The live activity monitoring dashboard shows the real-time information about NGINX Plus and the applications it is load balancing, which is helpful for troubleshooting. To access the dashboard, follow the steps from [here](/nginx-ingress-controller/logging-and-monitoring/status-page).
