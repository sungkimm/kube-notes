
# Namespace

## Create Namespace
``` bash
kubectl create namespace ${namespace}
```

## Create namespace via yml file
#### YAML file example
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace

```
``` bash
kubectl create -f namespace.ymal
```

## Create pod within namespace (via CLI)
``` bash
kubectl create -f filename.yml -n ${namespace}
```

## Add namespace inside yml file
#### Example
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test-namespace
```

## Set default namespace

#### View Current config
```bash
kubectl config view
```

#### Add context
```bash
kubectl config set-context $(kubectl config current-context) \
                                    --namespace=${namespace} 

# or
kubectl config set-context ${namespace} \
                    --cluster=kubernetes \
                    --user=kubernetes-admin \
                    --namespace=${namespace} 

# then check current context
kubectl config current-context
```




### Select context to set default namespace
```bash
kubectl config use-context ${namespace}
```



## Limit resources on namespace
#### Create ResourceQuota

```yaml
# yaml file
kind: ResourceQuota
```

