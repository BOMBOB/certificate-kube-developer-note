# Section 6 Pod Design

## Label, Selectors and Annotation

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
    name: simple-webapp
    labels: 
        app: App1
        function: Front-end
spec:
    containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
```
```yaml
# replicaset-definition.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: simple-webapp
    labels: 
        app: App1
        function: Front-end
    annotations:
        buildversion: 1.34
spec:
    replicas: 3
    selector:
        matchLabels:
            app: App1
    template:
        metadata:
            labels:
                app: App1
                function: Front-end
        spec:
            containers:
            - name: simple-webapp
              image: simple-webapp
```
Implementor  | Inspector
------------ 	| -------------
\- | k get pods --selector app=App1
\- | k get pods --show-labels
\- | k get pods -l env=dev
\- | k get pods -l env=dev --no-headers | wc -l
\- | k get all -l bu=finance --no-header | wc -l
\- | k get pods -l env=prod,bu=finance,tier=frontend

## Rollout and Versioning

> Deployment Strategy

>> Recreate: Destroy all and then create new ones instead.

>> Rolling Update: Destroy and create one by one.

`k apply -f deployment-definition.yml`

`k set image deployment/myapp-deployment nginx=nginx:1.9.1`

Action  | Command
------------ 	| -------------
Create | k create -f deployment-definition.yml
Get | k get deployments
Update | k apply -f deployment-definition.yml <br/> k set image deployment/myapp-deployment nginx=nginx:1.9.1
Status | k rollout status deployment/myapp-deployment <br/> k rollout history deployment/myapp-deployment
Rollback | k rollout undo deployment/myapp-deployment


Implementor  | Inspector
------------ 	| -------------
k rollout undo deployment/myapp-deployment | k rollout history deployment/myapp-deployment
\- | k rollout status deployment/myapp-deployment
\- | k get replicasets
