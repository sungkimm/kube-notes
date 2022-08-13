# Configmap

## Imperative
#### Create a configmap imperatively
``` bash
$ kubectl create configmap <config-name> --from-literal=<key>=<val> \ 
                                         --from-literal=<key>=<val>
```


## Declarative
#### Configmap YAML example
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-name
data:
  database_url: mongodb-service
  key: val
```


## To list configs
```bash
kubectl describe configmaps <configmap-name>
```


## Configuration with pod


#### An Example on how to inject an env variable to a container


**Configmap YAML example**
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  key: val
  key2: val2
  key3: val3
```


#### How to inject the entire env variables from configmap
**Pod YAML example**  
``` yaml
kind: Pod
...
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

#### How to inject only a single env variable from configmap
**Pod YAML example**
``` yaml
kind: Pod
...
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name:
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: key
```
Configmap must be run before pod run.  