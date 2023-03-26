---
layout: post
title: k8s autoscaling
date: 2023-03-16
tags: k8s autoscaling
---

300 秒 偵測每次加減一個 POD
```
kubectl apply -f - <<EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: metrics-server-v0.5.2
  namespace: kube-system
spec:
  behavior:
    scaleDown:
      policies:
      - periodSeconds: 300
        type: Percent
        value: 1
      selectPolicy: Min
    scaleUp:
      policies:
      - periodSeconds: 300
        type: Percent
        value: 1
      selectPolicy: Max
      stabilizationWindowSeconds: 0
  maxReplicas: 30
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: metrics-server-v0.5.2
EOF
```

kubectl get events -n kube-system
```
LAST SEEN   TYPE     REASON    OBJECT                                       MESSAGE
16m         Normal   Pulled    pod/metrics-server-v0.5.2-6bf845b67f-lwqtd   Container image "gke.gcr.io/metrics-server:v0.5.2-gke.3" already present on machine
16m         Normal   Created   pod/metrics-server-v0.5.2-6bf845b67f-lwqtd   Created container metrics-server
16m         Normal   Started   pod/metrics-server-v0.5.2-6bf845b67f-lwqtd   Started container metrics-server
42m         Normal   Pulled    pod/metrics-server-v0.5.2-6bf845b67f-t8dg9   Container image "gke.gcr.io/metrics-server:v0.5.2-gke.3" already present on machine
42m         Normal   Created   pod/metrics-server-v0.5.2-6bf845b67f-t8dg9   Created container metrics-server
42m         Normal   Started   pod/metrics-server-v0.5.2-6bf845b67f-t8dg9   Started container metrics-server
```
kubectl describe hpa metrics-server-v0.5.2 -n kube-system
```
Name:                                                  metrics-server-v0.5.2
Namespace:                                             kube-system
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Tue, 14 Mar 2023 15:42:22 +0800
Reference:                                             Deployment/metrics-server-v0.5.2
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  47% (23m) / 80%
Min replicas:                                          1
Max replicas:                                          30
Behavior:
  Scale Up:
    Stabilization Window: 0 seconds
    Select Policy: Max
    Policies:
      - Type: Percent  Value: 1  Period: 300 seconds
  Scale Down:
    Select Policy: Min
    Policies:
      - Type: Percent  Value: 1  Period: 300 seconds
Deployment pods:       2 current / 2 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    recommended size matches current size
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:           <none>

```
