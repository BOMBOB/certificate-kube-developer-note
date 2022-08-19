# Kube command

 [<a>==>Cheatsheet from kube website:</a>](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Getting start
```
* kubectl run hello-minikube
* kubectl cluster-info
* kubectl get nodes
```

## POD

Implementor  								 	| Inspector
------------------------------------  	| -------------
* kubectl run nginx --image nginx  		| * kubectl get pods
* 												| k get pods webapp
* 											 	| k get all

## Yaml in kube

```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels:
		app: myapp			# DFWYW(define whatever you want)
		type: front-end  	# DFWYW

spec:
	containers:
	- name: nginx-container # "-" is list
	  image: nginx
```
```bash

```
Implementor  								 	| Inspector
------------------------------------  	| -------------
 kubectl create -f pod-definition.yml  | kubectl get pods
 kubectl run nginx --image nginx			| kubectl describe pod newpods-wt78l
 k delete pod webapp 						| k get pods -o wide
 k apply -f pod.yaml						| k describe pod newpods-bcvm4 \| grep -i image
\*** k edit pod redis # after save, it will update pod auto 							| k run redis --image=redis123 --dry-run=client -o yaml \> pod.yaml
				\-							| kubectl get pod <pod-name> -o yaml > pod-definition.yaml
												
![](https://drive.google.com/uc?id=1MOeYLJGUlP3KUNtoT3zUUu-pgyrLvlBY)

## ReplicationController & ReplicaSet

### ReplicationController
ReplicationController span across multiple nodes in the cluster

```yaml
# rc-definition.yml
apiVersion: v1
kind: ReplicationController
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: front-end
spec: 
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
			spec:
				containers:
				- name: nginx-container
				  image: nginx
	replicas: 3
		
	
```
Implementor			| Inspector
-----------------   | --------------
k create -f rc-definition.yml | k get replicationcontroller

### ReplicaSet
```yaml
#replicaset-definition.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	labels: 
		app: myapp
		type: front-end
spec:
	template:
		metadata: 
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
			- name: nginx-controller
			  image: nginx
	replicas: 3
	selector: # ReplicationControler doesn't have this property (Optional)
		matchLabels: 
			type: front-end
	
```
Implementor			| Inspector
-----------------   | --------------
k create -f replicaset-definition.yml | k get replicaset # k get replicasets.apps
k replace -f replicaset-definition.yml # if you update replicas number | k describe replicasets.apps new-replica-set
k scale --replicas=6 -f replicaset-definition.yml | \-
k scale --replicas=6 replicaset my-replicaset | \-
k delete replicaset myapp-replicaset # Also deletes all underlying PODs | \-
k edit replicasets.apps new-replica-set | \-


## Deployment
![](https://shorturl.at/nrsV3)

```yaml
# deployment-definition.yml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
				
		spec:
			containers:
			- name: nginx-controller
			  image: nginx
	replcas: 3
	selector:
		matchLabels:
			type: front-end
		
```
Implementor			| Inspector
-----------------   | --------------
k create -f deployment-definition.yml 	| k get deployments
k apply -f deployment-definition.yml	 	| k describe deployments.apps frontend-deployment
k create deployment httpd-frontend --image=httpd:2.4-alpine | \-
k scale deployment --replicas=3 httpd-frontend | \-

## Namespace
namespace: kube-system, default and kube-public

real cases: dev, prod

Each namespace can define authorization.

dns: \<service-name\>.\<namespace\>.svc.cluster.local
![](https://drive.google.com/uc?id=1TA-Fjk2JYNBpNjrBbebP_ogLi_iYNmL_)


```yaml
# namepsace-dev.yml
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```
 ### ResourceQuota
 
```yaml
# compute-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namespace: dev
space:
	hard:
		pods: "10"
		requests.cpu: "4"
		request.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi
```


Implementor			| Inspector
-----------------   | --------------
k create -f pod-definition.yml --namespace=dev| k get pods --namespace=kube-sys
k create -f namespace-dev.yml | k config set-context ${k config current-context) --namespace=dev
k create namespace dev	| k get pods --all-namespaces
\- | k config view \| grep namespace # Show current namespace
\- | k get ns
\- | k get ns --no-headers | wc -l
\- | k -n research get pods --no-headers
\- | k -n dev get svc


## Imperative command


Implementor			| Inspector
-----------------   | --------------
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml | \-
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml | 
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml | \-
k expose pod redis --name redis-service --port 6379 --target-port 6379 | k describe svc redis-service 
k create deployment webapp --image=kodekloud/webapp-color | \-
k scale deployment --replicas=3 webapp | \-
k run customer-nginx --image=nginx --port 8080 # Export 8080 | \-
k create ns dev-ns | \-
k create deployment redis-deploy --image=redis --namespace=dev-ns --dry-run=client -o yaml > redis.yaml | k get deployments.apps -n dev-ns
k apply -f redis.yaml | \-
k run httpd --image=httpd:alpine --port 80 --expose --dry-run=client -o yaml # Create httpd with expose 80 | k get pod httpd

