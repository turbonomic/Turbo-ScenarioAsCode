# Namespace with Quota Scenario 1
This scenario creates a namespace with quota, a single deployment with 2 container replicas with limit of CPU traffic.

Expectation: Even though the CPU limit is not enough, the resize should take into account the namespace quota

## Create using all_in_one.yaml
```console
➜  namespaceQuota git:(master) ✗ kubectl create -f all_in_one.yaml
namespace/quotatest created
resourcequota/quotatest created
deployment.apps/cpug-30 created
```

This creates a namespace with quota of cpu.limit = 90m, and a deployment of 2 replicas with cpu.limit=10m each. Check the current quota usage.

```console
➜  namespaceQuota git:(master) ✗ kubectl get resourcequota quotatest -nquotatest --output=yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: "2020-04-29T02:23:52Z"
  name: quotatest
  namespace: quotatest
  resourceVersion: "40567622"
  selfLink: /api/v1/namespaces/quotatest/resourcequotas/quotatest
  uid: 62460fd4-9f89-48df-adac-bc7ba1ec62f7
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

Prior to Turbonomic 7.22.0, the namespace quota is not considered. We see two resize actions for each of the container to resize from 27MHz to 127MHz, which is around 10millicore to 47millicore. If we resize both of them, the resize action failed with the following error.

```console
I0429 02:18:38.331548       1 resize_container_util.go:215] Begin to resize container[quotatest/memg-30-654d7c576d-j4gz8-0] parent=ReplicaSet/memg-30-654d7c576d.
I0429 02:18:38.331640       1 resize_container_util.go:67] Try to update container mem-load resource limit from map[cpu:{i:{value:10 scale:-3} d:{Dec:<nil>} s:10m Format:DecimalSI} memory:{i:{value:41943040 scale:0} d:{Dec:<nil>} s: Format:BinarySI}] to map[cpu:{{48 -3} {<nil>} 48m DecimalSI} memory:{{41943040 0} {<nil>}  BinarySI}]
E0429 02:18:38.341875       1 resize_container_util.go:230] Failed to clone pod quotatest/memg-30-654d7c576d-j4gz8-0 with new size: pods "memg-30-654d7c576d-j4gz8-1c2h8qo47d3v1" is forbidden: exceeded quota: quotatest, requested: requests.memory=40Mi, used: requests.memory=80Mi, limited: requests.memory=100Mi
E0429 02:18:38.342023       1 resize_container.go:253] Failed to execute resize action: pods "memg-30-654d7c576d-j4gz8-1c2h8qo47d3v1" is forbidden: exceeded quota: quotatest, requested: requests.memory=40Mi, used: requests.memory=80Mi, limited: requests.memory=100Mi
E0429 02:18:38.342091       1 action_handler.go:200] Failed to execute action RIGHT_SIZE on CONTAINER [quotatest/memg-30-654d7c576d-j4gz8/mem-load]: pods "memg-30-654d7c576d-j4gz8-1c2h8qo47d3v1" is forbidden: exceeded quota: quotatest, requested: requests.memory=40Mi, used: requests.memory=80Mi, limited: requests.memory=100Mi
```
In Turbonomic 7.22.0, we introduced the support for namespace quota for resize. We should no longer see the resize action that violates the quota limit.
