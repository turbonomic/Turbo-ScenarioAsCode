# Namespace with Quota Scenario 3
This scenario creates a namespace with quota, 2 deployments each of which deploys 2 pod replicas, where each pod replica has one container with limit of memory traffic.

Expectation: The resize should take into account the namespace quota

## Create using all_in_one.yaml
```console
➜  namespaceQuota3 git:(master) ✗ kubectl apply -f all_in_one.yaml
namespace/nsquota-test-multi-deployments2 created
resourcequota/nsquota-test-multi-deployments2 created
deployment.apps/quota-test1 created
deployment.apps/quota-test2 created
```

This creates a namespace with quota of memory.limit=2200Mi, and 2 deployments each with 2 replicas with memory.limit=500Mi each. Check the current quota usage.

```console
➜  namespaceQuota3 git:(master) ✗ kubectl-pt-dc11 get resourcequota nsquota-test-multi-deployments2 -n nsquota-test-multi-deployments2 --output=yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ResourceQuota","metadata":{"annotations":{},"name":"nsquota-test-multi-deployments2","namespace":"nsquota-test-multi-deployments2"},"spec":{"hard":{"limits.memory":"2200Mi"}}}
  creationTimestamp: "2020-05-07T15:35:19Z"
  name: nsquota-test-multi-deployments2
  namespace: nsquota-test-multi-deployments2
  resourceVersion: "43396474"
  selfLink: /api/v1/namespaces/nsquota-test-multi-deployments2/resourcequotas/nsquota-test-multi-deployments2
  uid: 4b9a6475-636a-488b-b304-beb8ec811d35
spec:
  hard:
    limits.memory: 2200Mi
status:
  hard:
    limits.memory: 2200Mi
  used:
    limits.memory: 2000Mi
 ```

Prior to Turbonomic 7.22.0, the namespace quota is not considered. When memory utilization is very high, we see two resize actions for each of the container to resize from 0.488 GB (500 MB) to 0.551 GB (564 MB). If we resize all of them, the resize action of the containers in the second deployment will fail because new total limits will exceed the namespace limit quota

In Turbonomic 7.22.0, we introduced the support for namespace quota for resize. We should no longer see the resize action that violates the quota limit.
