# Windows Test Scenarios
These scenarios are to sanity test the support of windows nodes in a hybrid k8s cluster. 
The scenarios are classified in their respective folders and an unified dockerfile for the image used is kept in the top folder.
All the binaries or scripts that are used in the container are compiled in this dockerfile.
Additionally the names of the resources are matched with the names expected by the QE automated test suite for the respective test. 
The workloads use one of:
- A simple powershell script
- [Stress utility](https://github.com/dhoomakethu/stress) written in golang (a port of [CPULoadGenerator](https://github.com/GaetanoCarlucci/CPULoadGenerator) for cpu loads.
- [TestLimit utility](https://docs.microsoft.com/en-us/sysinternals/downloads/testlimit) for memory reservations.

The image needs to be built on a windows machine/vm, with compiled stress.exe in the same directory as that of the Dockerfile with
```
docker build -f Dockerfile -t irfdocker/powershell:latest .
```

## Simple metrics observation
This scenario creates a multiple containers per pod deployment with some cpu and memory usage and the requests and limits specified. The automated test suite compares the metrics observed on the containers and pods with those queried from the kubelet.

The yaml is kept under folder `monitoring`.

## Scaling
The yamls are meant to  generate scale actions in various combinations. The names are self explainatory.

The yamls are kept under folder `scaling`.

## Pod Move tests
The yaml is a simple arbitrary workload that uses some cpu and some memory, with limits in place.

For a pod move action a simple node label strategy need to be applied as below.

1. Apply and arbitrary label to one of the windows nodes
```
kubectl label node <node1-name> kubeturbo=pod-move
```

2. Create the deployment (which already have this label among the nodeSelectors)

3. Once the pod is created, remove the label from the first node and apply on to another windows nodes
```
kubectl label node ip-191-168-18-158.ec2.internal kubeturbo-
kubectl label node <node2-name> kubeturbo=pod-move
```
