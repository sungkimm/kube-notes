# Node Affinity

Node affinity feature is to ensure that pods are hosted on particular nodes. 


## Label Nodes
``` bash
kubectl label nodes <node-name> <label0key>=<label-value>

# ex)
kubectl label nodes node-1 size=Large
```

## Node Selector Pod Example YAML
Node selector is a simpler version of node affinity  
```yaml
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:
    size: Large
```

## Node Affinity Pod Example YAML
```yaml
spec:
  containers:
    - name: nginx
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values:
                - Large
      
      ############## OR ############
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
            - key: size
              operator: In
              values:
                - Large
```


### Attribute types

#### Operator
* `In`
* `NotIn`
* `Exists`
* `DoesNotExist`
* `Gt`
* `Lt`.

#### Other attributes

* `requiredDuringSchedulingIgnoredDuringExecution`
* `preferredDuringSchedulingIgnoredDuringExecution`


`DuringScheduling`: is the state where a pod does not exist and is created for the first time.  

`DuringExecution`: is the state where a pod has been running and a change is made in the environment that affects node affinity such as a change in that label of a node. 

#### Example 
`requiredDuringScheduling`: the scheduler will mandate the pod to be placed on a node with the given affinity rules

`preferredDuringScheduling`: where a matching node is not found, the scheduler will simply ignore affinity rules and place the pot on any available node. 


`IgnoredDuringExecution` means pod will continue to run and any chages in node affinity will not impact them once they are scheduled. 