# Executable Container Resizing Limited By Namespace Quota Constraint

This scenario is to test the executable part of container resizing actions when it's limited by namespace resource quota constraint.

```
Namespace:
- CPU limit quota capacity: 2000m

- ContainerSpec:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 2600m
```

Turbo analysis will eventually generate **executable** resizing up action from 500m to 1900m on this container. And namespace resizing will not be generated.
