# Section 5 Observability

## Readiness and Liveness Probes

### Readiness Probes
>HTTP Test -/api/ready

```yaml
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
	  readinessProbe:
	  	httpGet:
	  		path: /api/ready
	  		port: 8080
```
>TCP test -3306

```yaml
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
	  readinessProbe:
	  	tcpSocket:
	  		port:3306
```
>Exec Command

```yaml
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
	  readinessProbe:
	  	exec:
	  		command:
	  		  - cat
	  		  - /app/is_ready
```


### Liveness Probes
>HTTP Test -/api/ready

```yaml
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
	  livenessProbe:
	  	httpGet:
	  		path: /api/healthy
	  		port: 8080
	  	initialDelaySeconds: 10
	  	periodSeconds: 5
	  	failureThreshold: 8
	  	
```
>TCP test -3306

```yaml
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
	  livenessProbe:
	  	tcpSocket:
	  		port:3306
```
>Exec Command

```yaml
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
	  livenessProbe:
	  	exec:
	  		command:
	  		  - cat
	  		  - /app/is_ready
```

Implementor  								 	| Inspector
------------------------------------  	| -------------


## Logs

Implementor  								 	| Inspector
------------------------------------  	| -------------
k create -f event-simulator.yaml | k logs -f event-simulatorpod
\- | k logs -f \<pod\> \<container-name\> # MultipleContainer

## Monitor and Debug Application
Implementor  								 	| Inspector
------------------------------------  	| -------------
\- | k top node
\- | k top pod