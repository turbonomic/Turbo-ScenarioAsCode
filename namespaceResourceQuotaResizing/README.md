# Namespace Resource Quota Resizing Scenarios

This repo provides a set of scenarios to test namespace resource quota resizing.

## Background

As a kubernetes app owner, resource quota is usually defined by the cluster admin. Application owners need to interact with the cluster admin when running out of resource quota. It serves as an isolation, as well as resource management purpose in a multi-tenant environment.

Currently, in Turbo, when we resize container, we take into account the resource quota as well as the node utilization. As a containerized app owner, if my application is encountering a resource-related performance problem, I want to take the maximum effective action I can execute right now without having to interact with a separate team (cluster admin) and engage in a potentially lengthy process to increase my quota. However, if the application performance could still benefit from additional capacity beyond what the quota permits, I would also like to get an action to increase my quota which I can work with the cluster admin to resolve.