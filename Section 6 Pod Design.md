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

### Lab
`k create -f deployment-definition.yml`

`k rollout status deployment/myapp-deployment`

`k rollout history deployment/myapp-deployment`

`k delete deployment myapp-deployment`

`k get all`

`k create -f deployment-definition.yml --record`

`k rollout deployment/myapp-deployment`

`k rollout history deployment/myapp-deployment`

`k apply -f deployment-definition.yml`

`k rollout status deployment/myapp-deployment`

`k set image deployment/myapp-deployment nginx-container=nginx:1.12-perl`


## Job
```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
    name: math-pod
spec: 
    containers:
      - name: math-add
        image: ubuntu
        command: ['expr', '3', '+', '2']
    restartPolicy: Never
```
```yaml
#job-definition.yaml
apiVersion: batch/v1
kind: Job
metadata:
    name: math-add-job
spec: 
    completions: 3
    parallelism: 3
    template:
        spec: 
            containers:
              - name: math-add
                image: ubuntu
                command: ['expr', '3', '+', '2']
            restartPolicy: Never
```

` k create -f job-definition.yaml`

` k get jobs`

` k get pods`

` k logs math-add-job-1d87pn`

` k delete job math-add-job`

## Cronjob

```yaml
# cron-jbo-definition.yaml
apiVersion: vatch/v1beta1
kind: CronJob
metadata:
    name: reporting-cron-job
spec: # CronJob
    schedule: "*/1 * * * *"
    jobTemplate:
        spec: # Job
            completions: 3
            parallelism: 3
            template:
                spec: #POD
                    containers:
                    - name: math-add
                        image: ubuntu
                        command: ['expr', '3', '+', '2']
                    restartPolicy: Never

```

`k create -f cron-job-definition.yaml`

`k get cronjob`

### Lab

> k create job throw-dice-job --image kodekloud/throw-dice --dry-run=client -o yaml > job.yaml

```yaml
#job.yaml
apiVersion: batch/v1
kind: Job
metadata:
    name: throw-dice-job
spec:
    backoffLimit: 25
    completions: 3

    template:
        metadata: 
            creationTimestamp: null
        spec:
            containers:
              - image: kodekloud/throw-dice
                name: throw-dice-job
                resources: {}
            restartPolicy: Never
```
