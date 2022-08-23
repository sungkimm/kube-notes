# Taint and Tolerations

`taint` : restriction. Prevent pods from being placed on the node
`tolerance` : tolerant to taint. Pods with tolerance can be placed on the node.  
`intolerance` : by default pods have no tolerations. None of the pods can tolerate any taint. Pods cannot be placed on node with taint by default. 

**IMPORTANT**  
Taints and tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations.  


## How to taint node

```bash
kubectl taint nodes <node-name> key-value:<taint-effect>

ex)
kubectl taint nodes gpu=true:NoSchedule


# To remove the taint 
kubectl taint nodes node1 key1=value1:NoSchedule-
```

**taint-effect**
`NoSchedule`: The Kubernetes scheduler will only allow scheduling pods that have tolerations for the tainted nodes.  
`PreferNoSchedule`: try to avoid placing pod on the node, but not guaranteed.  
`NoExecute`: new pords will not be scheduled on the node and existing pod on the node, if any, will be evicted if they do not tolerate the taint(these pods may have been scheduled on the node before taint was applied to the node)


## How to add tolerations to pod


#### Example Pod YAML
All of these values need to be encoded in double qoutation. 
"Match" the taint created by the `kubectl taint`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
  
  tolerations:
    - key: "<taint key>"  #"gpu"
      operator: "Equal"
      value: "<taint value>" #"true"
      effect: "NoSchedule"
```
The default value for operator is Equal

A toleration "matches" a taint if the `keys` and `effect` are the same, and:

* the operator is `Exists` (in which case no value should be specified), or
* the operator is `Equal` and the `values` are equal.  


>Note:
>There are two special cases:
>
> 1. An empty key with operator Exists matches all keys, values and effects which means this will tolerate everything.  
> ex) **without a key, value, or effect** matches any taint in a node
> ```yaml
> tolerations:
>   operator: "Exists"
> ```
>
> 2. An empty effect matches all effects with key.  


`Exists` operator without `value` **matches any value of a Taint Key**  
We can match any value and effect in any node with the specified taint key by defining the operator and the key, as shown below.  
 ```yaml
 tolerations:
   operator: "Exists"
   key: "<taint key>"
 ```



## Taint on master node

Scheduler doest not schedule any pods on the master node. Taint is set on the master node. Automatically that prevents any pods from being scheduled on the master node. 


``` bash
kubectl describe node <master-node-name> | grep Taint


# output 
Taints: node-role.kubernetes.io/control-plane:NoSchedule
```



https://blog.kubecost.com/blog/kubernetes-taints/