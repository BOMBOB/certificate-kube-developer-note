

### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace.
```
k create ns mynamespace
k run nginx --image=nginx -n=mynamespace --restart=Never
```

### Create the pod that was just described using YAML
```
k run nginx --image=nginx -n=mynamespace --restart=Never --dry-run=client -o=yaml > definition.yaml
```
```
```