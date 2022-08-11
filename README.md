# Kubernetes Notes

Refer to a README file in each folder for further notes and example.


## Edit

``` bash
# edit a current running pod
kubectl edit pod pod-name
```

## Get output


``` bash
# use --dry-run=client to preview the object only(doesnt submit) then write info into yaml file
kubectl run redis --image=redis --dry-run=client -o yaml > pod-def.yml

# write a current running pod info into yaml file
kubectl get pod pod-name -o yaml > pod-def.yml
```



## Accessing a pod from another namespace
#### Example URL
```bash
db-service.dev.svc.cluster.local
```
`db-service` : service name(deployment, pod, etc)  
`dev` : namespace  
`svc` : service  
`cluster.local` : domain




## Labels and Selector
----
#### Label Definition at each point

LABEL-B and LABEL-C must match each other
``` yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx # LABEL-A: <--this label is to manage the deployment itself. this may be used to filter the deployment based on this label. 
spec:              
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx    #LABEL-B:  <--  this is where we tell replication controller to manage the pods. This field defines how the Deployment finds which Pods to manage. Must match with LABEL-C
  template:
    metadata:
      labels:
        app: my-nginx   #LABEL-C: <--this is the label of the pod, this must be same as LABEL-B
    spec:                
      containers:
      - name: my-nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "128Mi" #128 MB
            cpu: "200m" #200 millicpu (.2 cpu or 20% of the cpu)

```