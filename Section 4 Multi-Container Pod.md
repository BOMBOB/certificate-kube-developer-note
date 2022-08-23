# Section 4 Multi-Container Pod

> Ambassador

> Adapter

> Sidecar 

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp
	labels:
		name: simple-webapp
		
spec:
	containers:
	- name: simple-webapp
	  image: simple-webapp
	  ports:
	    - containerPort: 8080
	- name: log-agent
	  image: log-agent
```

 
Implementor  								 	| Inspector
------------------------------------  	| -------------
k run yellow --image=busybox --restart=Never --dry-run -o yaml > pod.yaml | k -n elastic-stack get pod,svc
\- | k -n elastic-stack logs app

```yaml
# pod.yaml //Sidecar
apiVersion: v1
kind: Pod
metadata:
	labels: 
		name: app
	name: app
	namespace: elastic-stack
spec:
	containers:
	- image: kodkloud/event-simulator
	  name: app
	  volumeMounts:
	  - mountPath: /log
	    name: log-volume
	- image: kodkloud/filebeat-configured
   	  name: sidecar
     volumeMounts:
     - mountPath: /var/log/event-simulator/
       name: log-volume
	volumes:
	- hostPath:
	 	path: /var/log/webapp
	 	type: DirectoryOrCreate
	  name: log-volume
	
```

## InitContainer

```yaml
apiVersion: v1
kind: Pod
metadata: 
	name: red
	namespace: default
	
spec:
	initContainers:
	- image: busybox
	  name: busybox
	  command: ["sleep", "1"]
	containers:
	- image: busybox
	  name: red-container
	  command:
	  - sh
	  - -c
	  - echo the app is running! && sleep 3600

```
Implementor  								 	| Inspector
------------------------------------  	| -------------
k replace --force -f pod.yaml