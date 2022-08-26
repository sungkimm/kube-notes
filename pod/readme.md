# Pod

1. Multi-Container Pod
2. Pod lifecyle
3. Liveness Probe
4. Init container
5. Infra container(pause)
6. Pod command and args
7. Static pod
8. Pod resource control
9. Pod env
10. Pod pattern
11. Security context


## 1. Multi-Container Pod
---

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
    - name: nginx
      image: nginx
    - name: centos
      image: centos:7
```

### How to access to container 
```bash
kubectl exec multipod -it -c contos -- /bin/bash
```

## 2. Pod lifecyle
### Pod lifecyle
---
By default, kubernetes controller re-run container if a task(container inside pod) fails.  

```yaml
spec:
  containers:
    ...
  restartPolicy: Always
```
### `restartPolicy` manages container, not pod

In kubernetes, `.spec.template.spec.restartPolicy = "Always"` is set as always by default. This attribute `restartPolicy` applies to **all containers** in the pod. After containers in a Pod exit, the kubelet restarts them by default.  


### When a container in pod **completed**
the container is re-run due to `.spec.template.spec.restartPolicy = "Always"`.  

### When a container in pod is **failed**  
A container in a Pod may fail for a number of reasons. ex) process exited with error code, memory exceeding, etc.

If this happens, and the `.spec.template.spec.restartPolicy = "OnFailure`, then the Pod stays on the node, but the container is re-run.
Therefore, your program needs to handle the case when it is restarted locally, or else specify `.spec.template.spec.restartPolicy = "Never"`


If a container fails and `.spec.template.spec.restartPolicy = "Never"`, your container doesn't re-run, but pod exists. 

#### Example output: `.spec.template.spec.restartPolicy = "Never"`
```bash
$ kubectl get pod -o wide
NAME              READY   STATUS      RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
test              0/1     Error       0          3m37s   10.44.0.1   worker1   <none>           <none>
```

## 3. Livness Probe
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



## 4. init container
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


## 5. Infra container(pause)
It's a default container inside pod created by kubernetes.
It a container manges IP, host, etc..


``` bash
# check if infra container is up and ready on worker node
docker ps -a 
```

## 6. Pod command and args
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

# if you want to replace entrypoint then use --entrypoint option(this will overwrite entrypoint in dockerfile)
# assume sleep2.0 is a command
# then it will run sleep2.0 10
$ docker run --entrypoint sleep2.0 ubuntu 10
```


#### How ENTRYPOINT and CMD works in Pod

`args` corresponds to CMD in `docker run CMD`. This will overwrite CMD in Dockerfile.  
`command` corresponds to entrypoint option in `docker run --entrypoint`. This will override Dockerfile entrypoint

``` yaml
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep2.0"]  # same as entrypoint option in docker run. This will override dockerfile entrypoint
      args: ["10"]  # same as CMD in docker run. This will override dockerfile CMD
```
The above setup will run:  
```bash
# this will run sleep2.0 10
$ doker run --entrypoint sleep2.0 ubuntu 10
```

#### Another Example
``` bash
# Dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]

# yaml
command: ["--color", "green"]


# result
$ --color green
```
The `command` from yaml will overwrite ENTRYPOINT in Dockerfile and there is no argument section provided here. THerefore it only runs `--color green`


## 7. Static pod
Static pod runs by kubelet daemon, not by API server from master node.  
Place pod yaml file under staticPodPath at worker node. 

Static pod path:  
``` bash
# save yaml file under
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# /etc/kubernetes/manifests
```
#### How to delete static pod
``` bash
# delete yml file
rm /etc/kubernetes/manifests/pod-ex.yml
```
If you changed staticPodPath, then make sure to restart kubelet daemon.  
``` bash 
systemctl restart kubelet
```



## 8. Pod resource control

`request`: minimum amoount of resources required to create a container
`limits`: minimum amoount of resources required to create a container. if process dies due to out of resouce, then pod gets rescheduled. 

#### Example:
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-resource
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      resources:
        requests:
          cpu: 200m  # 200 milli core
          memory: 250Mi
        limits:
          cpu: 1 # num core
          memory: 500Mi # MiB or Gi

```
`memory`: kubernetes uses MiB(Mi) or Gi
`cpu`: kubernetes uses either # of core or milli-core

#### units
``` text
# memory
1MB == 1000kb
1MiB == 1024kib

# core
1 core == 1000m
```

If only specifies limits without requests, then requests is set with the same spec as limits, as a default 


## 9. Pod env

#### Example
``` yaml
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: NEW_VAR
          value: newval
```


## 10. Pod pattern

multi-container pod
1. Sidecar
2. Adapter
3. Ambassador


## 11. Security context

Security settings on the container level will override the settings on the pod.  
#### Example YAML
To apply security settings on a `pod level`
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  securityContext:  # add it under spec
    runAsUser: 1000
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
```


To apply security settings on a `container level`
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      securityContext:  # add it under container
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"] # capability are only supported at the container level
```