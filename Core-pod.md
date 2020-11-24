## Home

# Core Concepts 

### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace using imperative command

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx -n mynamespace
```

</p>
</details>

### Create a pod with image nginx called nginx2 on this namespace using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl run nginx2 --image=nginx --dry-run=client -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
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
    name: nginx2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml -n mynamespace
```
</p>
</details>

### Create a busybox pod (using imperative command) that runs the command "env" Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --command -it -- env # -it will help in seeing the output

Alternate Solution
# Just run it without -it
kubectl run busybox --image=busybox --command -- env
# and then, check its logs
kubectl logs busybox
```

</p>
</details>

### Create a busybox2 pod with image busybox(using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run busybox2 --image=busybox --dry-run=client -o yaml --command -- env > busybox2.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox2
  name: busybox2
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f busybox2.yaml
kubectl logs busybox2
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```
Alternatively 

```bash
kubectl get po -A
```
</p>
</details>

### Change pod nginx with image nginx:1.7.1. this pod does not exist in default namespace. (Observe that the container will be restarted as soon as the image gets pulled)

<details><summary>show</summary>
<p>

```bash
# Identify the namespace this pod is running by listing pods in all namespaces. and note nginx pod runs in mynamespace
kubectl get po -A
```

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1 -n mynamespace
kubectl describe po nginx  -n mynamespace # you will see an event 'Container will be killed and recreated'
kubectl get po nginx  -n mynamespace -w # watch it
```

*Note*: some time after changing the image, you should see that the value in the `RESTARTS` column has been increased by 1, because the container has been restarted, as stated in the events shown at the bottom of the `kubectl describe pod nginx -n mynamespace` command:

```
Events:
  Type    Reason     Age                From               Message
  ----    ------     ----               ----               -------
  Normal  Scheduled  34m                default-scheduler  Successfully assigned mynamespace/nginx to node01
  Normal  Pulling    34m                kubelet, node01    Pulling image "nginx"
  Normal  Pulled     33m                kubelet, node01    Successfully pulled image "nginx"
  Normal  Killing    66s                kubelet, node01    Container nginx definition changed, will be restarted
  Normal  Pulling    65s                kubelet, node01    Pulling image "nginx:1.7.1"
  Normal  Pulled     17s                kubelet, node01    Successfully pulled image "nginx:1.7.1"
  Normal  Created    12s (x2 over 33m)  kubelet, node01    Created container nginx
  Normal  Started    10s (x2 over 33m)  kubelet, node01    Started container nginx

```

*Note*: you can check pod's image by running

```bash
kubectl get po nginx -n mynamespace -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -n mynamespace -o wide # get the IP
# create a temp busybox pod access nginx pod IP using its default port 80
kubectl run busybox --image=busybox --rm -it -- wget -O- <POD IP>:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -n mynamespace -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### Get the nginx pod's YAML

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -n mynamespace -o yaml
# or
kubectl get po nginx -n mynamespace -oyaml
# or
kubectl get po nginx -n mynamespace --output yaml
# or
kubectl get po nginx -n mynamespace --output=yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx -n mynamespace
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -n mynamespace
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -n mynamespace -p
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -n mynamespace -- /bin/sh
```

</p>
</details>

### Create a busybox3 pod in default namespace that echoes 'hello world' and then exits 

<details><summary>show</summary>
<p>

```bash
kubectl run busybox3 --image=busybox -it -- echo 'hello world'
# or
kubectl run busybox3 --image=busybox -it -- /bin/sh -c 'echo hello world'

# Later you can also view the same in its log
kubectl logs busybox3
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>
# --rm removes(deletes) the pod after its interactive shell exits.

```bash
kubectl run busybox4 --image=busybox -it --rm -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginxenv pod in default namespace and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginxenv --image=nginx --env=var1=val1
# then
kubectl exec -it nginxenv -- env
# or
kubectl exec -it nginxenv -- sh -c 'echo $var1'
# or
kubectl describe po nginxenv | grep val1
# or
kubectl run nginxenv --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>

### Clean up the created resources.

<details><summary>show</summary>
<p>

```bash
# simply delete the namespace you created to delete all the resources created under this.
kubectl delete ns mynamespace

# For all the resources created in default, you had to list it comma separated or run it one by one.
kubectl delete pod <list all the pods comma separated>

# During Exam time to delete a running pod is significant. Use can use no wait option to force it for immediate delete.
kubectl delete pod <pod-name> --grace-period=0 --force

```

</p>
</details>
