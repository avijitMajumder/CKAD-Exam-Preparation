### Home

# Configuration | ConfigMaps

### Create a configmap named userconfig with values user=test,role=dev

<details><summary>show</summary>
<p>

```bash
kubectl create configmap userconfig --from-literal=user=test --from-literal=role=dev
```

</p>
</details>

### View the values of the above created configmap

<details><summary>show</summary>
<p>

```bash
kubectl get cm userconfig -o yaml
# or
kubectl describe cm userconfig
```

</p>
</details>

### Create and display a configmap named serverconfig from a file

Create the file with

```bash
echo -e "host=localhost\nport=8080" > serverconfig.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm serverconfig --from-file=serverconfig.txt
kubectl get cm serverconfig -o yaml
```

</p>
</details>

### Create and display a configmap named appconfig from the same serverconfig.txt file, giving the key 'appserver'

<details><summary>show</summary>
<p>

```bash
kubectl create cm appconfig --from-file=appserver=serverconfig.txt
kubectl describe cm appconfig
kubectl get cm appconfig -o yaml
```

</p>
</details>

### Create and display a configmap named envconfig from a .env file

Create the file with the command

```bash
echo -e "# this is a comment\nvar1=val1\n# this is a another comment\nvar2=val2" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm envconfig --from-env-file=config.env
kubectl get cm envconfig -o yaml
```

</p>
</details>

### Create a new nginx pod that loads the value from variable 'var2' in above created envconfig to an env variable called 'variable2'

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
vi nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env:
    - name: variable2 # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: envconfig # name of config map
          key: var2 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f nginx.yaml
# View the env values inside the container as below which will show 'variable2=val2'
kubectl exec nginx -it -- env | grep variable2 
```

</p>
</details>

### Create a nginx2 pod with image nginx that loads the envconfig as env variables

<details><summary>show</summary>
<p>

```bash
kubectl run nginx2 --image=nginx -o yaml --dry-run=client > nginx2.yaml
vi nginx2.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx2
  name: nginx2
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx2
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: envconfig # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f nginx2.yaml
kubectl exec nginx2 -it -- env 
```

</p>
</details>

### Create a nginx3 pod with image nginx that loads the envconfig as a volume inside pod on path '/etc/data'. Create the pod and 'ls' into the '/etc/data' directory.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx3 --image=nginx -o yaml --dry-run=client > nginx3.yaml
vi nginx3.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx3
  name: nginx3
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: envconfig # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx3
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/data # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f nginx3.yaml
kubectl exec nginx3  -it -- /bin/sh
ls /etc/data # will show var1 var2
cat /etc/data/var1 # will show val1
cat /etc/data/var2 # will show val2
```

</p>
</details>
