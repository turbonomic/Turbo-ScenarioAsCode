# Namespace Resource Quota Resizing With Multiple Container Specs

Namespace has 2 container specs, each with one container. One has CPU highly utilized and the other has CPU under utilized:

```
Namespace:
- CPU limit quota capacity: 1000m
- CPU limit quota used: 1000m

- ContainerSpec1:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 800m
- ContainerSpec2:
    - Container:
        - CPU limit: 500m
        - Desired CPU limit from Turbo analysis: 300m
```

- With namespace reisizing support, 
    - in plan, as we simulate all container resizing actions, namespace quota resizing action will **NOT** be generated.
    - in real time, we'll generate following actions:
```
- Resize namespace from 1000m to 1300m
- Resize container1 from 500m to 800m (not executable)
```
    