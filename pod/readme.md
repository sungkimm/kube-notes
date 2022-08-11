# Pod

1. liveness Probe
2. init container
3. infra container(pause)


## 1. Livness Probe
---
#### Type:
```yaml
livenessProbe:
    httpGet:
        path: /
        port: 8080 
    tcpSocket:
        port: 22
    exec:
        command: 
        - ls 
        - command
```


#### Example:
``` yaml
spec:
  containers:
    - name: unhealthy-pod
      image: smlinux/unhealthy  
      ports:
        - containerPort: 8080
          protocol: TCP
      livenessProbe:
        httpGet: # checked by http request. if it doesnt get 200 code then restart the container
          path: /
          port: 8080 
        
        initialDelaySeconds: 5 # after N second container runs
        periodSeconds: 1 # how often request
        timeoutSeconds: 1 # response timeout
        successThreshold: 1
        failureThreshold: 1

        tcpSocket:
          port: 22
        
        exec:
          command: # command to run inside container, if 0 return then healthy
          - ls 
          - command
```



## 2. init container
----
One container depends on the other. If init container doesn't run properly then the main container doesn't run. 
The main container runs after init container is ready.


#### Example
``` yaml
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]

```


## 3. infra container(pause)
It's a default container inside pod created by kubernetes.
It a container manges IP, host, etc..


``` bash
# check if infra container is up and ready on worker node
docker ps -a 
```
