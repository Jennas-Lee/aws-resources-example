# Horizontal Pod Autoscaling

## Install the Kubernetes Metrics Server

``` shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get deployment metrics-server -n kube-system
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)

## Using HPA

### `autoscaling/v2`

``` yaml title="hpa.yaml" hl_lines="4 5 7 9 10 14 17 18 19 20 21" linenums="1"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 40
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
```

### `autoscaling/v1`

``` yaml title="hpa.yaml" hl_lines="4 5 7 9 10 11 13 14 15" linenums="1"
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 40
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)