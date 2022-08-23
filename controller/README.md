# Controller

Controller ensures that the specified number of pod replicas are running at any point of time. 



1. Replication Controller
2. ReplicaSet
3. Deployment(Rolling Update)
4. DaemonSet
5. StatefulSet
6. Job
7. CronJob  



## 1. Replication Controller

#### ReplicationController YAML Example

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rep
spec:
  replicas: 3
  selector:
    app: webui              # must match with one of pod labels
    version: "2.1"  # can be multiple, if multiple labels then selector uses "AND" condition
  template:                 # pod template
    metadata:
      name: nginx-pod
      labels:               # pod labels
        app: webui      
        version: "2.1"   # can have multiple lables
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

#### Edit number of scales

```bash
kubectl edit rc <rc-name>
# or 
kubectl scale rc <rc-name> --replicas=N
```

#### Delete Replication Controller

```bash 
# this will delete the entire running pod..
kubectl delete rc <rc-name>
# --cascade=false will keep pods alive
kubectl delete rc <rc-name> --cascade=false
```

## 2. ReplicaSet

Almost identical to replication controller, but provides diverse selector options.  


#### ReplicaSet YAML Example
``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
    matchExpressions:
      - {key: version, operator: In, values: ["2.1", "2.2"]}
    # - {key: version, operator: Exists }
  template:                 # pod template
    metadata:
      name: nginx-pod
      labels:               # pod labels
        app: webui      
        version: "2.1"   # can have multiple lables
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

`Operator` type:  
* In
* NotIn
* Exists: if key matches, include every values
* DoesNotExist
  



## 3. Deployment(Rolling update)

Deployment controls `ReplicaSet`  

`Deployment` -> `ReplicaSet` -> `Pod`  

Rolling update & Rollback

`Deployment` is exactly same as `ReplicaSet` unless you define `rollingupdate`.  

#### Deployment YAML Example

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webui
  template:                 # pod template
    metadata:
      name: nginx-pod
      labels:               # pod labels
        app: webui      
        version: "2.1"   # can have multiple lables
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

#### How Deployment works example  

`Deployment` -> `ReplicaSet` -> `Pod`  
```bash
$ kubectl get deploy,rs,pod    
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-dep   3/3     3            3           30s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-dep-546b4b7cbc   3         3         3       30s

NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-dep-546b4b7cbc-d29qt   1/1     Running   0          30s
pod/nginx-dep-546b4b7cbc-f7sw9   1/1     Running   0          30s
pod/nginx-dep-546b4b7cbc-q75cg   1/1     Running   0          30s

```

#### Delete Deployment

```bash 
# this will delete ONLY deployment
kubectl delete deployments <dp-name>
# To delete corresponding ReplicaSet, delete rs 
kubectl delete rs <rs-name> # --cascade=false
```

#### Delete ReplicaSet
If `ReplicaSet` is deleted, then `Deployment` will re-schedule `ReplicaSet` again, then `ReplicaSet` will create `pod`.   


#### Delete everything at once
```bash 
# this will delete deploy, rs, pod at once
kubectl delete -f path/to/dep.yml
```

## Rolling update & Rolling Back(Deployment)

Run deployment with `--record` option to keep history.  
``` bash
kubectl create -f path/to/deploy.yml --record
```

#### Rolling update
`set` command help you make changes to existing application resources.  
```bash
kubectl set image deployment <deploy-name> <container-name>=<new-ver-image> --record
```

#### Rolling-update status check
Check status of rolling-update while rolling-update is being executed.  
```bash
kubectl rollout status deployment <deploy-name>
```

#### Rolling-update pause/resume
Pause/resume rolling-update while rolling-update is being executed.  
```bash
kubectl rollout pause deployment <deploy-name>
# resume
kubectl rollout resume deployment <deploy-name>
```


Rollout history check  
```bash
kubectl rollout history deployment <deploy-name>

# ex) 
deployment.apps/nginx-dep
REVISION  CHANGE-CAUSE
1         kubectl create --filename=controller/deployment.yml --record=true
2         kubectl set image deployment nginx-dep web=nginx:1.15 --record=true
3         kubectl set image deployment nginx-dep web=nginx:1.16 --record=true
4         kubectl set image deployment nginx-dep web=nginx:1.17 --record=true

kubectl rollout undo deploy <deploy-name>
```



## 4. DaemonSet
## 5. StatefulSet
## 6. Job
## 7. CronJob

