# Namespace Resource Quota Resizing With Single Container

Namespace has one container with CPU congested:

```
Namespace:
- CPU limit quota capacity: 90m
- CPU limit quota used: 10m

ContainerSpec:
- Container:
    - CPU limit: 10m
    - Desired CPU limit to assure performance from Turbo analysis: 100m
```

- Without resizing up namespace CPU limit quota, no action will be generated on this container due to the quota constraint.
- With namespace reisizing support, we'll generate following actions:
```
- Resize namespace from 90m to 100m
- Resize container from 10m to 100m (not executable)
```