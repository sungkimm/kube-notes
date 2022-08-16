# Secret & Service account

## Imperative method

``` bash
kubectl create secret generic <secret-name> --from-literal=<key>=<value> \
                                           --from-literal=<key>=<value>

kubectl create secret generic <secret-name> --from-file=<path-to-file>
```


## Declarative method

``` bash
kubectl create -f 
```
#### Example YAML
Specify data in Hash format
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ex
data:
    DB_HOST: mysql
    DB_USER: LWIgdGV4dAo=
```

#### How to convert text to encoded format
```bash
echo -b "text" | base64
# LWIgdGV4dAo=
```

#### How to decode hash to text
```bash
echo -b "LWIgdGV4dAo" | base64 --decode
# text 
```


## How to view secrets

``` bash
kubectl get secrets
# see the attrubytes in the secrets, but hides the value
kubectl describe secrets
# to see the value in encoded format
kubectl get secrets <secret> -o yaml
```


## Configurate secret with pod


#### Injecting the entire secret as files
**Pod YAML**
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
    envFrom:
        - secretRef:
            name: <secret-name>
```

#### Injecting a single env variable
**Pod YAML**
``` yaml
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
    env:
        - name: DB_USER
            valueFrom: 
                SecretKeyRef:
                    name: <secret-name>
                    key: DB_USER
```


#### Secrets in pod as volumes
**Pod YAML**
``` yaml
volumes:
    - name: app-secret-vol
      secret :
        secretName: <secret-name>
```




## Service Account


#### Create service account

```bash
kubectl create serviceaccount <serviceaccount-name>
```

#### To list service account and token
``` bash
$ kubectl get serviceaccount 

NAME      SECRETS   AGE
default   0         8d

# list token inside service account
$ kubectl describe serviceaccount <serviceaccount-name>

# To view token 
kubectl describe secret <token-name>
```
Token can be used as an authentication bearer token while making a rest call to the kubernetest.  


#### Configurate service account with pod & Default service account 
Kubernetes automatically mounts the default service account by default. 

``` bash

$ kubectl get serviceaccount 

NAME      SECRETS   AGE
default   0         8d


$ kubectl describe pod <pod-name>
...

Mounts:
  /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wqdcw (ro)
```

Default token only has permissiob to run basic kubernetes API queries. 


``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
  serviceAccountName: <sa-name>  # to include a service account

  automountServiceAccountToken: false  # to disable default service account token

```



Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

Not checking-in secret object definition files to source code repositories.

Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD. 



Also the way kubernetes handles secrets. Such as:

A secret is only sent to a node if a pod on that node requires it.

Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.

Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the protections and risks of using secrets here



Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault. I hope to make a lecture on these in the future.