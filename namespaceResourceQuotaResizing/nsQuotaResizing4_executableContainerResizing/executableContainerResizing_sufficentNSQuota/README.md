# Executable Container Resizing With Sufficient Namespace Quota

This scenario is to test the executable part of container resizing actions when namespace has sufficient resource quota.

```
Namespace:
- CPU limit quota capacity: 5000m

- ContainerSpec:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 2600m
```

Turbo analysis will generate **executable** resizing up action from 500m to 2600m on this container
