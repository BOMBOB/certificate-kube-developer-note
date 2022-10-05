# Service & Networking

## Service

```yaml
#service-definition.yml
apiVersion: v1
kind: Service
metadata:
    name: myapp-service
spec:
    type: NodePort
    ports:
      - targetPort: 80
        port:80
        nodePort: 30008 # 30000 - 32767
    selector:
        app: myapp
        type: front-end
```
 > k create -f service-definition.yml
 
 > k get svc


 ## Cluster IP

 ```yaml
 #clusterip-definition.yml
apiVersion: v1
kind: Service
metadata: 
    name: back-end
spec:
    type: ClusterIP
    ports:
      - targetPort: 80
        port: 80
    selector:
        app: myapp
        type: back-end
```


## Ingress

```yaml
# ingress-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-ingress-controller
spec: 
    replica: 1
    selector:
        matchLabels:
            name: nginx-ingress

    template:
        metadata:
            labels: 
                name: nginx-ingress
        spec:
            containers:
              - name: nginx-ingress-controller
                image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
            args:
              - /nginx-ingress-controller
              - --configmap=$(POD_NAMESPACE)/nginx-configuration

            env:
              - name: POD_NAME
                valueFrom:
                    fieldRef:
                        fieldPath: metadata.name
              - name: POD_NAMESPACE
                valueFrom:
                    fieldRef:
                        fieldPath: metadata.namespace
            ports:
              - name: http
                containerPort: 80
              - name: https
                containerPort: 443 
```

```yaml
# config-map-nginx-configuration.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: nginx-configuration
```
```yaml
apiVersion: v1
kind: Service
metadata: 
    name: nginx-ingress

spec: 
    type: NodePort
    ports:
      - port: 80
        targetPort: 80
        protocol: TCP
        name: http
      - port: 443
        targetPort: 443
        protocol: TCP
        name: https
    selector:
        name: nginx-ingress
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata: 
    name: nginx-ingress-serviceaccount
```

### Create Ingress resource
```yaml
# Ingress-*wear.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear
spec
    backend:
        serviceName: wear-service
        servicePort: 80
```
> k create -f Ingress-wear.yaml

> k get ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear-watch
spec:
    rules:
      - http:
            paths:
              - path: /wear
                backend:
                    service:
                        name: wear-service
                        port:
                            number: 80
              - path: /watch
                backend:
                    service:
                        name: watch-service
                        port:
                            number: 80
```

> k describe ingress ingress-wear-watch

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: ingress-wear-watch
spec:
    rules:
      - host: wear.my-online-store.com
        http:
            paths:
              - backend:
                    serviceName: wear-service
                    servicePort: 80
      - host: watch.my-online-store.com
        http:
            paths:
              - backend:
                    serviceName: watch-service
                    servicePort: 80
```

Format - kubectl create ingress \<ingress-name\> --rule="host/path=service:port"

Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"