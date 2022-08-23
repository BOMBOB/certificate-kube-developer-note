# Section 3 Configuration

```docker
# docker run ubuntu-sleeper sleep 10 ## to override CMD command
FROM  ubuntu
CMD ["sleep", "5"]
```

```docker
# docker run ubuntu-sleeper 10 ## Pass the value
FROM  ubuntu
ENTRYPOINT ["sleep"]
```
```docker
# docker run ubuntu-sleeper 10 ## Pass the value and default is 5
# docker run --entrypoint sleep2.0 ubuntu-sleeper 10 ## override entrypoint
FROM  ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

## Command & Arguments


```yaml
# pod-definition.yml
# docker run --name ubuntu-sleeper ubuntu-sleeper
# docker run --name ubuntu-sleeper ubuntu-sleeper 10
# k create -f pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
	name: ubuntu-sleeper-pod
spec:
	containers:
	- name: ubuntu-sleeper
	  image: ubuntu-sleeper
	  args: ["10"]
```

```yaml
# pod-definition.yml
# docker run --entrypoint sleep2.0 ubuntu-sleeper 10 ## override entrypoint
# k create -f pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
	name: ubuntu-sleeper-pod
spec:
	containers:
	- name: ubuntu-sleeper
	  image: ubuntu-sleeper
	  command: ["sleep2.0"] # --entrypoint in docker world
	  args: ["10"] # last word in docker command
```

>Edit a POD Remember, you CANNOT edit specifications of an existing POD other than the below.

>spec.containers[*].image

>spec.initContainers[*].image

>spec.activeDeadlineSeconds

>spec.tolerations

### Lab

Implementor  								 	| Inspector
------------------------------------  	| -------------
k run webapp-green --image=kodekloud/webapp-color --dry-run -o yaml > pod.yaml | \-
k run webapp-green --image=kodekloud/webapp-color --restart=Never --dryrun -o yaml > pod.yaml | \-
```yaml
apiVersion: v1
kind: Pod
metadata:
	creationTimeStamp: null
	labels:
		run: webapp-green
	name: webapp-green
spec:
	containers:
	- image: kodekloud/webapp-color
	  name: webapp-green
	  args: ["--color=green"]
```

## Environment Variables

> Plain Key Value

> ConfigMap

> Secrets

### Plain Key Value 

docker run -e APP_COLOR=pink simple-webapp-color

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	  	- containerPort:8080
	  
	  env:
	  	- name: APP_COLOR
	  	  value: pink
```

### ConfigMap

```bash
k create configmap \
	app-config --from-literal=APP_COLOR=blue
k create configmap \
	app-config --from-file=app_config.properties
```
Implementor  								 	| Inspector
------------------------------------  	| -------------
k create configmap app-config --from-literal=APP_COLOR=blue | k get configmaps
k create configmap app-config --from-file=app_config.properties | k describe configmaps
\- | k get cm
\- | k explain pods --recu
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: app-config
data:
	APP_COLOR: blue
	APP_MODE: prod
```
```yaml
# mysql-config
apiVersion: v1
kind: ConfigMap
metadata:
	name: mysql-config
data:
	port: 3306
	max_allowed_packet: 128M
```

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	  envFrom:
	    - configMapRef:
	    		name: app-config
	volumes:
	- name: app-config-volume
	  configMap:
	  	name: app-config
```
```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	volumes:
	- name: app-config-volume
	  configMap:
	  	name: app-config
```
```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	  env:
	    - name: APP_COLOR
	      valueFrom:
	      		configMapKeyRef:
	      		  	name: app-config
	      		  	key: APP_COLOR
```

#### Lab
Implementor  								 	| Inspector
------------------------------------  	| -------------
\- | k explain pods --recursive | grep envFrom -A3


### Secret
Implementor  								 	| Inspector
------------------------------------  	| -------------
k create secret generic app-secret --from-literal=DB_HOST=mysql | k get secrets
k create secret generic app-secret --from-file=app_secret.properties | k describe secrets
\- | k get secret app-secret -o yaml

```yaml
# secret-data.yaml
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data
	DB_Host: bXlzcWw= # echo -n 'mysql' | base64
	DB_User: cm9vdA== # echo -n 'root' | base64
	DB_Password:cGFzd3Jk # echo -n 'password' | base64
```
> *** decode: echo -n 'bXlzcWw=' | base64 --decode

>ENV

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	  envFrom:
	    - secretRef:
	     		name: app-secret
```

>SINGLE ENV

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	  env
	    - name: DB_Password
	      valueFrom:
	      		secretKeyRef:
	      			name: app-secret
	      			key: DB_Password
```

>VOLUME

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp-color
	labels:
		name: simple-webapp-color
spec:
	containers:
	- name: simple-webapp-color
	  image: simple-webapp-color
	  ports:
	    - containerPort:8080
	volumes:
	- name: app-secret-volume
	  secret:
	  	secretName: app-secret
```
> ls /opt/app-secret-volumes

#### Lab
 
Implementor  								 	| Inspector
------------------------------------  	| -------------
\- | k get pods,svc
\- | k explain pods --recursive \| less


## Security
