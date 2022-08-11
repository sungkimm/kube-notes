# Pod

1. liveness Probe
2. init container
3. infra container(pause)
4. Pod command and args

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

## 4. Pod command and args
#### How ENTRYPOINT and CMD works in docker
##### 1. CMD
`CMD` gets overwritten by docker run CMD parameters

``` Dockerfile
# Dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

```bash
# this will do sleep 10 because command gets overwritten
$ docker run ubuntu sleep 10
```
##### 2. ENTRYPOINT 
`ENTRYPOINT` gets appended by docker run CMD parameters

``` Dockerfile
# Dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
```
```bash
# this will do sleep 10 because command gets appended
$ docker run ubuntu 10

# if cmd parameter is missing then error: sleep 
$ docker run ubuntu 
```
##### 3. Combination
``` Dockerfile
# Dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```bash
# if not specifiy cmd parameter them, sleep 5
$ docker run ubuntu

# if specifiy cmd parameter them, sleep 10
$ docker run ubuntu 10

# if you want to replace entrypoint then use --entrypoint option
# assume sleep2.0 is a command
# then it will run sleep2.0 10
$ docker run --entrypoint sleep2.0 ubuntu 10
```


#### How ENTRYPOINT and CMD works in Pod

`args` corresponds to CMD in `docker run CMD`  
`command` corresponds to entrypoint option in `docker run --entrypoint`

``` yaml
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep2.0"]  # same as entrypoint option in docker run
      args: ["10"]  # same as CMD in docker run
```
The above setup will run:  
```bash
# this will run sleep2.0 10
$ doker run --entrypoint sleep2.0 ubuntu 10
```

      