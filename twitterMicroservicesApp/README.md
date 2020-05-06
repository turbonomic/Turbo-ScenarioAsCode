# Twitter Microservices App
This scenario deploys a home grown twitter-like microservices application in an Istio service mesh.

The following is the architecture of the twitter application. The source code is hosted [here](https://github.com/turbonomic/demoapp).

<img width="717" alt="testbed" src="https://user-images.githubusercontent.com/12261551/45770729-2f5ead80-bc11-11e8-9d38-26394aabd63b.png">

## Deploy Istio 1.5+
We deploy Istio 1.5+ to take advantage of the [Telemetry V2 Istio standard metrics](https://istio.io/docs/reference/config/telemetry/metrics/). 
With Telemetry V2, Istio standard metrics are directly exported by the Envoy proxy. There are many more labels being
defined with V2 standard metrics than those from previous Istio version, so there is no need to define custom metrics any more
for our twitter application scenario.

* Recommend install with `istioctl`. Download from [here](https://istio.io/docs/setup/install/istioctl/).
* Install with either `default` or `demo` profile. The `demo` profile is the same as `default` plus the following addon components: `grafana`, `kiali` and `tracing`:
```
$ istioctl manifest apply --set profile=demo
```
## Deploy demoapp
* Clone the [demoapp](https://github.com/turbonomic/demoapp) repository:
```
$ git clone https://github.com/turbonomic/demoapp.git
```
* The installation will deploy everything in the `default` namespace. Enable automatic sidecar injection in the `default` namespace:
```
$ kubectl label namespace default istio-injection=enabled
```
* Make sure your cluster has a default storage class:
```
$ kubectl get sc
NAME            PROVISIONER             AGE
gp2 (default)   kubernetes.io/aws-ebs   116d
```
* The `demoapp` requires `cassandra` to be deployed. Generate dependent charts:
```
$ cd demoapp/deploy
$ helm dependency update demoapp
``` 
* Install the demoapp
```
$ helm template demoapp | kubectl create -f -
```
* By default, the following are installed:
  * Cassandra (3 replicas)
  * Locust (1 master, 1 worker)
  * TwitterApp (1 twitter-cass-api, 1 twitter-cass-tweet, 1 twitter-cass-friend, 1 twitter-cass-user)
```
meng@Mengs-MBP$ kubectl-awslemur get po
NAME                                  READY   STATUS    RESTARTS   AGE
cassandra-0                           3/3     Running   0          38h
cassandra-1                           3/3     Running   0          38h
cassandra-2                           3/3     Running   0          38h
locust-master-7bbb558f8c-smgdh        2/2     Running   0          36h
locust-worker-7dc647c84f-6vnj8        2/2     Running   0          36h
twitter-cass-api-65765d4fd8-7cn6t     2/2     Running   0          22h
twitter-cass-friend-c5f9cdcff-9f9b4   2/2     Running   0          35h
twitter-cass-tweet-6488c6454f-4zg7p   2/2     Running   0          34h
twitter-cass-user-6d447bbbf4-pzzx6    2/2     Running   0          35h
```
* **NOTE**: `Cassandra` takes much longer to start and become ready. You probably need to restart all `twitter-cass-*` pods
after `Cassandra` becomes ready to make sure all components of the twitter app can establish connection to the DB.

## Run the simulation
* Connect to the `locust-master` service
```
$ kubectl get svc locust-master
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP                                                                  PORT(S)                                        AGE
locust-master   LoadBalancer   10.107.49.255   adffd8a1f615c4ec7ace77aaf3cd5fbd-1341603673.ca-central-1.elb.amazonaws.com   8089:32388/TCP,5557:30680/TCP,5558:30872/TCP   7d18h
```
* Specify the number of users and hatch rate and start the traffic. 

## Configure traffic pattern
By default, the traffic pattern is 100%, 10% with 30 min interval. To change that:
* Edit the `demoapp/values.yaml`, and modfiy the following values:
```
  time_slot_probability: 1.0,0.1
  time_slot_probability_step_in_min: 30
```
* Wait for Locust pods to restart, and restart the traffic.
