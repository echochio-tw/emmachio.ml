---
layout: post
title: GKE 安裝 redis, minio
date: 2022-12-09
tags: k8s GKE
---

需設定
節點集區詳細資料 中繼資料 Kubernetes 標籤 type=egame

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: standard-rwo
EOF

helm repo add stable https://charts.helm.sh/stable
helm repo update
kubectl create namespace major
helm install redis stable/redis -f redis.yaml -n major
helm install minio stable/minio -f minio.yaml -n major
```

```
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass
  redis: {}

## Bitnami Redis image version
## ref: https://hub.docker.com/r/bitnami/redis/tags/
##
image:
  registry: docker.io
  repository: bitnami/redis
  ## Bitnami Redis image tag
  ## ref: https://github.com/bitnami/bitnami-docker-redis#supported-tags-and-respective-dockerfile-links
  ##
  #tag: 5.0.7-debian-10-r32
  tag: 6.0.3
  ## Specify a imagePullPolicy
  ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
  ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
  ##
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

## String to partially override redis.fullname template (will maintain the release name)
##
# nameOverride:

## String to fully override redis.fullname template
##
# fullnameOverride:

## Cluster settings
cluster:
  enabled: false
  slaveCount: 1

## Use redis sentinel in the redis pod. This will disable the master and slave services and
## create one redis service with ports to the sentinel and the redis instances
sentinel:
  enabled: false
  ## Require password authentication on the sentinel itself
  ## ref: https://redis.io/topics/sentinel
  usePassword: true
  ## Bitnami Redis Sentintel image version
  ## ref: https://hub.docker.com/r/bitnami/redis-sentinel/tags/
  ##
  image:
    registry: docker.io
    repository: bitnami/redis-sentinel
    ## Bitnami Redis image tag
    ## ref: https://github.com/bitnami/bitnami-docker-redis-sentinel#supported-tags-and-respective-dockerfile-links
    ##
    tag: 6.0.3 #5.0.7-debian-10-r27
    ## Specify a imagePullPolicy
    ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
    ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
    ##
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName
  masterSet: mymaster
  initialCheckTimeout: 5
  quorum: 2
  downAfterMilliseconds: 60000
  failoverTimeout: 18000
  parallelSyncs: 1
  port: 26379
  ## Additional Redis configuration for the sentinel nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Enable or disable static sentinel IDs for each replicas
  ## If disabled each sentinel will generate a random id at startup
  ## If enabled, each replicas will have a constant ID on each start-up
  ##
  staticID: false
  ## Configure extra options for Redis Sentinel liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 5
  ## Redis Sentinel resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m
  ## Redis Sentinel Service properties
  service:
    ##  Redis Sentinel Service type
    type: ClusterIP
    sentinelPort: 26379
    redisPort: 6379
    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    # sentinelNodePort:
    # redisNodePort: 

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:

## Specifies the Kubernetes Cluster's Domain Name.
##
clusterDomain: cluster.local

networkPolicy:
  ## Specifies whether a NetworkPolicy should be created
  ##
  enabled: false

  ## The Policy model to apply. When set to false, only pods with the correct
  ## client label will have network access to the port Redis is listening
  ## on. When true, Redis will accept connections from any source
  ## (with the correct destination port).
  ##
  # allowExternal: true

  ## Allow connections from other namespacess. Just set label for namespace and set label for pods (optional).
  ##
  ingressNSMatchLabels: {}
  ingressNSPodMatchLabels: {}

serviceAccount:
  ## Specifies whether a ServiceAccount should be created
  ##
  create: false
  ## The name of the ServiceAccount to use.
  ## If not set and create is true, a name is generated using the fullname template
  name:

rbac:
  ## Specifies whether RBAC resources should be created
  ##
  create: false

  role:
    ## Rules to create. It follows the role specification
    # rules:
    #  - apiGroups:
    #    - extensions
    #    resources:
    #      - podsecuritypolicies
    #    verbs:
    #      - use
    #    resourceNames:
    #      - gce.unprivileged
    rules: []

## Redis pod Security Context
securityContext:
  enabled: true
  fsGroup: 1001
  runAsUser: 1001
  ## sysctl settings for master and slave pods
  ##
  ## Uncomment the setting below to increase the net.core.somaxconn value
  ##
  # sysctls:
  # - name: net.core.somaxconn
  #   value: "10000"

## Use password authentication
usePassword: false
## Redis password (both master and slave)
## Defaults to a random 10-character alphanumeric string if not set and usePassword is true
## ref: https://github.com/bitnami/bitnami-docker-redis#setting-the-server-password-on-first-run
##
password: ""
## Use existing secret (ignores previous password)
# existingSecret:
## Password key to be retrieved from Redis secret
##
# existingSecretPasswordKey:

## Mount secrets as files instead of environment variables
usePasswordFile: false

## Persist data to a persistent volume (Redis Master)
persistence: {}
  ## A manually managed Persistent Volume and Claim
  ## Requires persistence.enabled: true
  ## If defined, PVC must be created manually before volume will be bound
  # existingClaim:

# Redis port
redisPort: 6379

##
## Redis Master parameters
##
master:
  ## Redis command arguments
  ##
  ## Can be used to specify command line arguments, for example:
  ##
  command: "/run.sh"
  ## Additional Redis configuration for the master nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Redis additional command line flags
  ##
  ## Can be used to specify command line flags, for example:
  ##
  ## extraFlags:
  ##  - "--maxmemory-policy volatile-ttl"
  ##  - "--repl-backlog-size 1024mb"
  extraFlags: []
  ## Comma-separated list of Redis commands to disable
  ##
  ## Can be used to disable Redis commands for security reasons.
  ## Commands will be completely disabled by renaming each to an empty string.
  ## ref: https://redis.io/topics/security#disabling-of-specific-commands
  ##
  disableCommands:
  - FLUSHDB
  - FLUSHALL

  ## Redis Master additional pod labels and annotations
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
  podLabels: {}
  podAnnotations: {}

  ## Redis Master resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m
  ## Use an alternate scheduler, e.g. "stork".
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  # schedulerName:

  ## Configure extra options for Redis Master liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 5
  nodeSelector: 
    type: egame

  ## Redis Master Node selectors and tolerations for pod assignment
  ## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
  ## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#taints-and-tolerations-beta-feature
  ##
  # nodeSelector: {"beta.kubernetes.io/arch": "amd64"}
  # tolerations: []
  ## Redis Master pod/node affinity/anti-affinity
  ##
  # affinity:
  #   nodeAffinity:
  #     requiredDuringSchedulingIgnoredDuringExecution:
  #       nodeSelectorTerms:
  #       - matchExpressions:
  #         - key: type
  #           operator: In
  #           values:
  #           - egame


  ## Redis Master Service properties
  service:
    ##  Redis Master Service type
    type: NodePort
    port: 6379

    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    nodePort: 30030

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:
    # loadBalancerSourceRanges: ["10.0.0.0/8"]

  ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    enabled: true 
    ## The path the volume will be mounted at, useful when using different
    ## Redis images.
    path: /data
    ## The subdirectory of the volume to mount to, useful in dev environments
    ## and one PV for multiple services.
    subPath: ""
    ## redis data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    storageClass: "standard-rwo"
    accessModes:
    - ReadWriteOnce
    size: 8Gi
    ## Persistent Volume selectors
    ## https://kubernetes.io/docs/concepts/storage/persistent-volumes/#selector
    matchLabels: {}
    matchExpressions: {}

  ## Update strategy, can be set to RollingUpdate or onDelete by default.
  ## https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets
  statefulset:
    updateStrategy: RollingUpdate
    ## Partition update strategy
    ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
    # rollingUpdatePartition:

  ## Redis Master pod priorityClassName
  # priorityClassName: {}

##
## Redis Slave properties
## Note: service.type is a mandatory parameter
## The rest of the parameters are either optional or, if undefined, will inherit those declared in Redis Master
##
slave:
  ## Slave Service properties
  service:
    ## Redis Slave Service type
    type: ClusterIP
    ## Redis port
    port: 6379
    ## Specify the nodePort value for the LoadBalancer and NodePort service types.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    # nodePort:

    ## Provide any additional annotations which may be required. This can be used to
    ## set the LoadBalancer service type to internal only.
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    annotations: {}
    labels: {}
    loadBalancerIP:
    # loadBalancerSourceRanges: ["10.0.0.0/8"]

  ## Redis slave port
  port: 6379
  ## Can be used to specify command line arguments, for example:
  ##
  command: "/run.sh"
  ## Additional Redis configuration for the slave nodes
  ## ref: https://redis.io/topics/config
  ##
  configmap:
  ## Redis extra flags
  extraFlags: []
  ## List of Redis commands to disable
  disableCommands:
  - FLUSHDB
  - FLUSHALL

  ## Redis Slave pod/node affinity/anti-affinity
  ##
  affinity: {}

  ## Configure extra options for Redis Slave liveness and readiness probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    successThreshold: 1
    failureThreshold: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    timeoutSeconds: 10
    successThreshold: 1
    failureThreshold: 5

  ## Redis slave Resource
  # resources:
  #   requests:
  #     memory: 256Mi
  #     cpu: 100m

  ## Redis slave selectors and tolerations for pod assignment
  # nodeSelector: {"beta.kubernetes.io/arch": "amd64"}
  # tolerations: []

  ## Use an alternate scheduler, e.g. "stork".
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  # schedulerName:

  ## Redis slave pod Annotation and Labels
  podLabels: {}
  podAnnotations: {}

  ## Redis slave pod priorityClassName
  # priorityClassName: {}

  ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    enabled: false
    ## The path the volume will be mounted at, useful when using different
    ## Redis images.
    path: /data
    ## The subdirectory of the volume to mount to, useful in dev environments
    ## and one PV for multiple services.
    subPath: ""
    ## redis data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # storageClass: "-"
    accessModes:
    - ReadWriteOnce
    size: 8Gi
    ## Persistent Volume selectors
    ## https://kubernetes.io/docs/concepts/storage/persistent-volumes/#selector
    matchLabels: {}
    matchExpressions: {}

  ## Update strategy, can be set to RollingUpdate or onDelete by default.
  ## https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets
  statefulset:
    updateStrategy: RollingUpdate
    ## Partition update strategy
    ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
    # rollingUpdatePartition:
## Prometheus Exporter / Metrics
##
metrics:
  enabled: false

  image:
    registry: docker.io
    repository: bitnami/redis-exporter
    tag: 1.4.0-debian-10-r3
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName

  ## Metrics exporter resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  ##
  # resources: {}

  ## Extra arguments for Metrics exporter, for example:
  ## extraArgs:
  ##   check-keys: myKey,myOtherKey
  # extraArgs: {}

  ## Metrics exporter pod Annotation and Labels
  podAnnotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9121"
  # podLabels: {}

  # Enable this if you're using https://github.com/coreos/prometheus-operator
  serviceMonitor:
    enabled: false
    ## Specify a namespace if needed
    # namespace: monitoring
    # fallback to the prometheus default unless specified
    # interval: 10s
    ## Defaults to what's used if you follow CoreOS [Prometheus Install Instructions](https://github.com/helm/charts/tree/master/stable/prometheus-operator#tldr)
    ## [Prometheus Selector Label](https://github.com/helm/charts/tree/master/stable/prometheus-operator#prometheus-operator-1)
    ## [Kube Prometheus Selector Label](https://github.com/helm/charts/tree/master/stable/prometheus-operator#exporters)
    selector:
      prometheus: kube-prometheus

  ## Custom PrometheusRule to be defined
  ## The value is evaluated as a template, so, for example, the value can depend on .Release or .Chart
  ## ref: https://github.com/coreos/prometheus-operator#customresourcedefinitions
  prometheusRule:
    enabled: false
    additionalLabels: {}
    namespace: ""
    rules: []
      ## These are just examples rules, please adapt them to your needs.
      ## Make sure to constraint the rules to the current postgresql service.
      #  - alert: RedisDown
      #    expr: redis_up{service="{{ template "redis.fullname" . }}-metrics"} == 0
      #    for: 2m
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} down
      #      description: Redis instance {{ "{{ $instance }}" }} is down.
      #  - alert: RedisMemoryHigh
      #    expr: >
      #       redis_memory_used_bytes{service="{{ template "redis.fullname" . }}-metrics"} * 100
      #       /
      #       redis_memory_max_bytes{service="{{ template "redis.fullname" . }}-metrics"}
      #       > 90 =< 100
      #    for: 2m
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} is using too much memory
      #      description: Redis instance {{ "{{ $instance }}" }} is using {{ "{{ $value }}" }}% of its available memory.
      #  - alert: RedisKeyEviction
      #    expr: increase(redis_evicted_keys_total{service="{{ template "redis.fullname" . }}-metrics"}[5m]) > 0
      #    for: 1s
      #    labels:
      #      severity: error
      #    annotations:
      #      summary: Redis instance {{ "{{ $instance }}" }} has evicted keys
      #      description: Redis instance {{ "{{ $instance }}" }} has evicted {{ "{{ $value }}" }} keys in the last 5 minutes.


  ## Metrics exporter pod priorityClassName
  # priorityClassName: {}
  service:
    type: ClusterIP
    ## Use serviceLoadBalancerIP to request a specific static IP,
    ## otherwise leave blank
    # loadBalancerIP:
    annotations: {}
    labels: {}

##
## Init containers parameters:
## volumePermissions: Change the owner of the persist volume mountpoint to RunAsUser:fsGroup
##
volumePermissions:
  enabled: false
  image:
    registry: docker.io
    repository: bitnami/minideb
    tag: buster
    pullPolicy: Always
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName
  resources: {}
  # resources:
  #   requests:
  #     memory: 128Mi
  #     cpu: 100m

## Redis config file
## ref: https://redis.io/topics/config
##
configmap: |-
  # Enable AOF https://redis.io/topics/persistence#append-only-file
  appendonly yes
  # Disable RDB persistence, AOF persistence already enabled.
  save ""

## Sysctl InitContainer
## used to perform sysctl operation to modify Kernel settings (needed sometimes to avoid warnings)
sysctlImage:
  enabled: false
  command: []
  registry: docker.io
  repository: bitnami/minideb
  tag: buster
  pullPolicy: Always
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName
  mountHostSys: false
  resources: {}
  # resources:
  #   requests:
  #     memory: 128Mi
  #     cpu: 100m

## PodSecurityPolicy configuration
## ref: https://kubernetes.io/docs/concepts/policy/pod-security-policy/
##
podSecurityPolicy:
  ## Specifies whether a PodSecurityPolicy should be created
  ##
  create: false

```

```
## Provide a name in place of minio for `app:` labels
##
nameOverride: ""

## Provide a name to substitute for the full names of resources
##
fullnameOverride: ""

## set kubernetes cluster domain where minio is running
##
clusterDomain: cluster.local

## Set default image, imageTag, and imagePullPolicy. mode is used to indicate the
##
image:
  repository: minio/minio
  tag: RELEASE.2020-06-14T18-32-17Z
  pullPolicy: IfNotPresent

## Set default image, imageTag, and imagePullPolicy for the `mc` (the minio
## client used to create a default bucket).
##
mcImage:
  repository: minio/mc
  tag: RELEASE.2020-05-28T23-43-36Z
  pullPolicy: IfNotPresent

## Set default image, imageTag, and imagePullPolicy for the `jq` (the JSON
## process used to create secret for prometheus ServiceMonitor).
##
helmKubectlJqImage:
  repository: bskim45/helm-kubectl-jq
  tag: 3.1.0
  pullPolicy: IfNotPresent

## minio server mode, i.e. standalone or distributed.
## Distributed Minio ref: https://docs.minio.io/docs/distributed-minio-quickstart-guide
##
mode: standalone

## Additional arguments to pass to minio binary
extraArgs: []

## Update strategy for Deployments
DeploymentUpdate:
  type: RollingUpdate
  maxUnavailable: 0
  maxSurge: 100%

## Update strategy for StatefulSets
StatefulSetUpdate:
  updateStrategy: RollingUpdate

## Pod priority settings
## ref: https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/
##
priorityClassName: ""

## Set default accesskey, secretkey, Minio config file path, volume mount path and
## number of nodes (only used for Minio distributed mode)
## AccessKey and secretKey is generated when not set
## Distributed Minio ref: https://docs.minio.io/docs/distributed-minio-quickstart-guide
##
existingSecret: ""
accessKey: "AKIAIOSFODNN7EXAMPLE"
secretKey: "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
certsPath: "/etc/minio/certs/"
configPathmc: "/etc/minio/mc/"
mountPath: "/export"

## Override the root directory which the minio server should serve from.
## If left empty, it defaults to the value of {{ .Values.mountPath }}
## If defined, it must be a sub-directory of the path specified in {{ .Values.mountPath }}
bucketRoot: ""

# Number of drives attached to a node
drivesPerNode: 1
# Number of MinIO containers running
replicas: 4
# Number of expanded MinIO clusters
zones: 1

## TLS Settings for Minio
tls:
  enabled: false
  ## Create a secret with private.key and public.crt files and pass that here. Ref: https://github.com/minio/minio/tree/master/docs/tls/kubernetes#2-create-kubernetes-secret
  certSecret: ""
  publicCrt: public.crt
  privateKey: private.key

## Enable persistence using Persistent Volume Claims
## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
##
persistence:
  enabled: true

  ## A manually managed Persistent Volume and Claim
  ## Requires persistence.enabled: true
  ## If defined, PVC must be created manually before volume will be bound
  existingClaim: ""

  ## minio data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  ## Storage class of PV to bind. By default it looks for standard storage class.
  ## If the PV uses a different storage class, specify that here.
  storageClass: "standard-rwo"
  ##VolumeName: ""
  accessMode: ReadWriteOnce
  size: 2Gi

  ## If subPath is set mount a sub folder of a volume instead of the root of the volume.
  ## This is especially handy for volume plugins that don't natively support sub mounting (like glusterfs).
  ##
  subPath: ""

## Expose the Minio service to be accessed from outside the cluster (LoadBalancer service).
## or access it from within the cluster (ClusterIP service). Set the service type and the port to serve it.
## ref: http://kubernetes.io/docs/user-guide/services/
##

service:
  type: NodePort
  clusterIP: ~
  port: 9000
  nodePort: 32010

  ## List of IP addresses at which the Prometheus server service is available
  ## Ref: https://kubernetes.io/docs/user-guide/services/#external-ips
  ##
  externalIPs: []
  #   - externalIp1

  annotations: {}
    # prometheus.io/scrape: 'true'
    # prometheus.io/path:   '/minio/prometheus/metrics'
    # prometheus.io/port:   '9000'

## Configure Ingress based on the documentation here: https://kubernetes.io/docs/concepts/services-networking/ingress/
##

imagePullSecrets: []
# - name: "image-pull-secret"

ingress:
  enabled: false
  labels: {}
    # node-role.kubernetes.io/ingress: platform

  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
    # kubernetes.io/ingress.allow-http: "false"
    # kubernetes.io/ingress.global-static-ip-name: ""
    # nginx.ingress.kubernetes.io/secure-backends: "true"
    # nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    # nginx.ingress.kubernetes.io/whitelist-source-range: 0.0.0.0/0
  path: /
  hosts:
    - chart-example.local
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

## Node labels for pod assignment
## Ref: https://kubernetes.io/docs/user-guide/node-selection/
##
nodeSelector: 
  type: egame

## Add stateful containers to have security context, if enabled MinIO will run as this
## user and group NOTE: securityContext is only enabled if persistence.enabled=true
securityContext:
  enabled: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000

# Additational pod annotations
podAnnotations: {}

# Additional pod labels
podLabels: {}

## Liveness and Readiness probe values.
## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
livenessProbe:
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 1
readinessProbe:
  initialDelaySeconds: 30
  periodSeconds: 5
  ## Set this to 1s higher than MINIO_API_READY_DEADLINE
  timeoutSeconds: 6
  successThreshold: 1
  failureThreshold: 3

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    cpu: 2m
    memory: 10Mi

## Create a bucket after minio install
##
defaultBucket:
  enabled: false
  ## If enabled, must be a string with length > 0
  name: bucket
  ## Can be one of none|download|upload|public
  policy: none
  ## Purge if bucket exists already
  purge: false

## Create multiple buckets after minio install
## Enabling `defaultBucket` will take priority over this list
##
buckets: []
  # - name: bucket1
  #   policy: none
  #   purge: false
  # - name: bucket2
  #   policy: none
  #   purge: false

## Additional Annotations for the Kubernetes Batch (make-bucket-job)
makeBucketJob:
  annotations:

## Additional Annotations for the Kubernetes Batch (update-prometheus-secret)
updatePrometheusJob:
  annotations:

s3gateway:
  enabled: false
  replicas: 4
  serviceEndpoint: ""
  accessKey: ""
  secretKey: ""

## Use minio as an azure blob gateway, you should disable data persistence so no volume claim are created.
## https://docs.minio.io/docs/minio-gateway-for-azure
azuregateway:
  enabled: false
  # Number of parallel instances
  replicas: 4

## Use minio as GCS (Google Cloud Storage) gateway, you should disable data persistence so no volume claim are created.
## https://docs.minio.io/docs/minio-gateway-for-gcs

gcsgateway:
  enabled: false
  # Number of parallel instances
  replicas: 4
  # credential json file of service account key
  gcsKeyJson: ""
  # Google cloud project-id
  projectId: ""

ossgateway:
  enabled: false
  # Number of parallel instances
  replicas: 4
  endpointURL: ""

## Use minio on NAS backend
## https://docs.minio.io/docs/minio-gateway-for-nas

nasgateway:
  enabled: false
  # Number of parallel instances
  replicas: 4
  # For NAS Gateway, you may want to bind the PVC to a specific PV. To ensure that happens, PV to bind to should have
  # a label like "pv: <value>", use value here.
  pv: ~

## Use minio as Backblaze B2 gateway
## https://github.com/minio/minio/blob/master/docs/gateway/b2.md
b2gateway:
  enabled: false
  # Number of parallel instances
  replicas: 4

## Use this field to add environment variables relevant to Minio server. These fields will be passed on to Minio container(s)
## when Chart is deployed
environment:
  MINIO_API_READY_DEADLINE: "5s"
  ## Please refer for comprehensive list https://docs.minio.io/docs/minio-server-configuration-guide.html

networkPolicy:
  enabled: false
  allowExternal: true

## PodDisruptionBudget settings
## ref: https://kubernetes.io/docs/concepts/workloads/pods/disruptions/
##
podDisruptionBudget:
  enabled: false
  maxUnavailable: 1

## Specify the service account to use for the Minio pods. If 'create' is set to 'false'
## and 'name' is left unspecified, the account 'default' will be used.
serviceAccount:
  create: true
  ## The name of the service account to use. If 'create' is 'true', a service account with that name
  ## will be created. Otherwise, a name will be auto-generated.
  name:

metrics:
  # Metrics can not be disabled yet: https://github.com/minio/minio/issues/7493
  serviceMonitor:
    enabled: false
    additionalLabels: {}
    # namespace: monitoring
    # interval: 30s
    # scrapeTimeout: 10s

## ETCD settings: https://github.com/minio/minio/blob/master/docs/sts/etcd.md
etcd:
  endpoints: []
  pathPrefix: ""
  corednsPathPrefix: ""
  clientCert: ""
  clientCertKey: ""
```
