# Twitter Microservices App
This scenario deploys a home grown twitter-like microservices application in an Istio service mesh.

The following is the architecture of the twitter application. The source code is hosted [here](https://github.com/turbonomic/demoapp).

<img width="717" alt="testbed" src="https://user-images.githubusercontent.com/12261551/45770729-2f5ead80-bc11-11e8-9d38-26394aabd63b.png">

## Deploy Istio 1.5
We deploy Istio 1.5 to take advantage of the [Telemetry V2 Istio standard metrics](https://istio.io/docs/reference/config/telemetry/metrics/). 
With Telemetry V2, Istio standard metrics are directly exported by the Envoy proxy. There are many more labels being
defined with V2 standard metrics than those from previous Istio version, so there is no need to define custom metrics any more
for our twitter application scenario.
