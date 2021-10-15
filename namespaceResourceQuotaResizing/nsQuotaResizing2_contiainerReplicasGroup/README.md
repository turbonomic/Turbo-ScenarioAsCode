# Namespace Resource Quota Resizing With Container Replicas Group

Namespace has one container spec with 2 replicas, where both containers have CPU congested:

```
Namespace:
- CPU limit quota capacity: 150m
- CPU limit quota used: 20m

ContainerSpec:
- Container1:
    - CPU limit: 10m
    - Desired CPU limit from Turbo analysis: 100m
- Container2:
    - CPU limit: 10m
    - Desired CPU limit from Turbo analysis: 100m
```

- Without resizing up namespace CPU limit quota, although one single container resizing up to 100m is allowed, 2 container replicas resizing up is not permitted due to the quota constraint.
- With namespace reisizing support, we'll generate following actions:
```
- Resize namespace from 150m to 200m
- Resize both containers from 10m to 100m (not executable)
```
