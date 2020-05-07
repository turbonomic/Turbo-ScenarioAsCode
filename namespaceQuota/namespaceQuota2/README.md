# Namespace with Quota Scenario 2
This scenario creates a namespace with quota, 2 deployments each of which deploys single pod with a single container with limit of CPU traffic.

Expectation: The resize should take into account the namespace quota

## Create using all_in_one.yaml
```console
➜  namespaceQuota2 git:(master) ✗ kubectl apply -f all_in_one.yaml
namespace/nsquota-test-multi-deployments created
resourcequota/nsquota-test-multi-deployments created
deployment.apps/quota-test1 created
deployment.apps/quota-test2 created
```

This creates a namespace with quota of cpu.limit=90m, and 2 deployments each with 1 replica with cpu.limit=10m each. Check the current quota usage.

```console
➜  namespaceQuota2 git:(master) ✗ kubectl get resourcequota nsquota-test-multi-deployments -n nsquota-test-multi-deployments --output=yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ResourceQuota","metadata":{"annotations":{},"name":"nsquota-test-multi-deployments","namespace":"nsquota-test-multi-deployments"},"spec":{"hard":{"limits.cpu":"90m","limits.memory":"150Mi","requests.cpu":"1","requests.memory":"100Mi"}}}
  creationTimestamp: "2020-05-01T22:13:59Z"
  name: nsquota-test-multi-deployments
  namespace: nsquota-test-multi-deployments
  resourceVersion: "41509395"
  selfLink: /api/v1/namespaces/nsquota-test-multi-deployments/resourcequotas/nsquota-test-multi-deployments
  uid: a9788ec1-5a88-4be5-8bae-6aae645b5627
spec:
  hard:
    limits.cpu: 90m
    limits.memory: 150Mi
    requests.cpu: "1"
    requests.memory: 100Mi
status:
  hard:
    limits.cpu: 90m
    limits.memory: 150Mi
    requests.cpu: "1"
    requests.memory: 100Mi
  used:
    limits.cpu: 20m
    limits.memory: 80Mi
    requests.cpu: 20m
    requests.memory: 80Mi
 ```

Prior to Turbonomic 7.22.0, the namespace quota is not considered. When CPU utilization is very high, we see two resize actions for each of the container to resize from 27MHz to 127MHz, which is around 10millicore to 47millicore. If we resize both of them, the resize action of the second container will fail because new total limits will exceed the namespace limit quota

In Turbonomic 7.22.0, we introduced the support for namespace quota for resize. We should no longer see the resize action that violates the quota limit.
