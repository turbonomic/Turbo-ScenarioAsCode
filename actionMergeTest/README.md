# Action Merge Test Scenario
This scenario creates a namespace with a deployment with 2 pod replicas. Each pod has 2 containers (an application container and a fake sidecar) and each container is underutilized for CPU and memory, where Turbo will recommend resizing down actions for these resources.

Expectation: With the improvement of action merge, we'll be able to merge all container resizing changes on WorkloadController, which represents a K8s controller, in this scenario, a deployment.

## Create using all_in_one.yaml
```console
➜  actionMergeTest git:(master) ✗ kubectl create -f all_in_one.yaml
namespace/action-merge-test created
deployment.apps/action-merge-test created
```

This creates a namespace `action-merge-test` with a deployment `action-merge-test` with replicas=2. Each pod has 2 containers:

1. Container `cpu-mem-load` has following resources over-provisioned:
```yaml
resources:
  limits:
    memory: 200Mi
    cpu: 200m
  requests:
    memory: 130Mi
    cpu: 100m
```

2. Container `cpu-mem-load-test-sidecar` has following resources over-provisioned:
```yaml
resources:
  limits:
    memory: 200Mi
  requests:
    memory: 130Mi
```

## Action merge
Each container is under-utilized, so it's expected that Turbo will recommend resizing down cpu/memory limits/request for container `cpu-mem-load` and resizing down memory limits/request for container `cpu-mem-load-test-sidecar`. The actions are shown as follows:
![Individual container resizing actions](./screenshots/individual%20container%20resizing%20actions.png)

### Before action merge improvement
There are in total 12 individual container resizing actions (the top one in the screenshot above is a merged action on Workload Controller which we'll talk about later). To execute all actions, there would be 6 restarts on each of the 2 pod replicas of this deployment, especially in this scenario where there are 2 containers in the pod, which is very disruptive.

### After action merge improvement
All individual container actions are merged onto one deployment:
![Merged action on WorkloadController](./screenshots/merged%20action%20on%20workload%20controller.png)
There will be only one restart on each of the 2 pod replicas of this deployment to make all corresponding changes to set the desired resources on the containers (`cpu-mem-load` and `cpu-mem-load-test-sidecar`).
