# Executable Container Resizing

This scenario is to test the executable part of container resizing actions due to namespace resource quota constraint.

## Container Executable Resizing Up to the Best Desired Value

First create such namespace with very large resource quota (or without resource quota):

```
Namespace:
- CPU limit quota capacity: 5000m

- ContainerSpec:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 2600m
```

Turbo analysis will generate **executable** resizing up action from 500m to 2600m on this container

## Container Executable Resizing Up to the Best Value Permitted by NS Quota

Then reduce namespace resource quota to 2000m:
```
Namespace:
- CPU limit quota capacity: 2000m

- ContainerSpec:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 2600m
```

Turbo analysis will eventually generate **executable** resizing up action from 500m to 1900m on this container. And namespace resizing will not be generated.
