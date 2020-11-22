# This tutorial will cover the following topics:
## Basic Deployment Example
* terminationGracePeriodSeconds: 0
* Updating the image in a Pod
* maxUnavailable and maxSurge
* maxSurge: 5 maxUnavailable: 5
* maxSurge: 10 maxUnavailable: 10
## 1) Basic Deployment Example
nano myDeployment.yaml 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']
``` 


         
*template* defines the Pod we want
###### replicas defines we want 10 identical copies running 
###### strategy type: RollingUpdate - we will see how updates work later. ( RollingUpdate is the default value )
###### Note the second line above kind: Deployment
###### Create the Deployment
##### kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
##### We use the following command to show the status of our Deployment :
```
kubectl rollout status deployment.v1.apps/busybox-deployment

Waiting for deployment "busybox-deployment" rollout to finish: 7 of 10 updated replicas are available...
Waiting for deployment "busybox-deployment" rollout to finish: 8 of 10 updated replicas are available...
Waiting for deployment "busybox-deployment" rollout to finish: 9 of 10 updated replicas are available...
deployment "busybox-deployment" successfully rolled out
```
```
All 10 replicas we requested are created simultaneously.
We'll be using Busybox as our Linux operating system as it is small (only 2MB) and suitable for our applications.
To create a container in a Pod running busybox is very fast. If you took 10 seconds to cut and paste the previous command all you may see is the final success line.
Alternative command to get information about our Deployment status.
kubectl get deploy

NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
busybox-deployment   10        10        10           10          23s
kubectl rollout status shows detail per Pod - one line per Pod 
kubectl get deploy shows overall summary status - compressed in one summary line.
Over time you will realize when to use which one.
Get as much detail as we can about our Deployment:
kubectl describe deployment.extensions/busybox-deployment

Name:                   busybox-deployment
Namespace:              default
CreationTimestamp:      Sat, 19 Jan 2019 07:51:48 +0200
Labels:                 app=busybox
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=busybox
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=busybox
  Containers:
   busybox:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      echo Container 1 is Running ; sleep 3600
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   busybox-deployment-668d59f6b7 (10/10 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  68s   deployment-controller  Scaled up replica set busybox-deployment-668d59f6b7 to 10
Note the useful Replicas: 10 desired | 10 updated | 10 total | 10 available | 0 unavailable line.
Show details about individual running Pods:
kubectl get pods --show-labels

NAME                                  READY   STATUS    RESTARTS   AGE    LABELS
busybox-deployment-668d59f6b7-762nm   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-9sglb   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-f5728   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-gzwbr   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-j8b9g   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-nszns   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-sm86q   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-w99jh   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-xgbfq   1/1     Running   0          3m6s   app=busybox,pod-template-hash=668d59f6b7
PS C:\k8>
Show a history of this Deployment, right now just one revision. Later in this tutorial we will have and use more revisions.
This is our first revision: there is no CHANGE-CAUSE. More about this later.
kubectl rollout history deployment.v1.apps/busybox-deployment

deployment.apps/busybox-deployment

REVISION  CHANGE-CAUSE
1         <none>
The purpose of this part of tutorial was to create one basic Deployment and introduce you to several commands that enable you to get details out your Deployments.
This Deployment served its purpose, delete the Deployment
kubectl delete -f myDeployment.yaml --force --grace-period=0

deployment.apps "busybox-deployment" force deleted
Show status of Pods during the delete phase:
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE     LABELS
busybox-deployment-668d59f6b7-762nm   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-9sglb   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-f5728   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-gzwbr   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-j8b9g   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-nszns   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-sm86q   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-w99jh   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-xgbfq   1/1     Terminating   0          6m13s   app=busybox,pod-template-hash=668d59f6b7
... several seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE     LABELS
busybox-deployment-668d59f6b7-762nm   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-9sglb   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-f5728   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-gzwbr   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-j8b9g   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-nszns   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-sm86q   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-w99jh   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-xgbfq   1/1     Terminating   0          6m17s   app=busybox,pod-template-hash=668d59f6b7
... several seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE     LABELS
busybox-deployment-668d59f6b7-762nm   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-9sglb   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-f5728   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-gzwbr   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-j8b9g   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-nszns   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-sm86q   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-w99jh   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-xgbfq   1/1     Terminating   0          6m26s   app=busybox,pod-template-hash=668d59f6b7
... several annoying seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE     LABELS
busybox-deployment-668d59f6b7-762nm   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-9sglb   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-f5728   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-gzwbr   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-j8b9g   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-nszns   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-sm86q   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-w99jh   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-xgbfq   1/1     Terminating   0          6m41s   app=busybox,pod-template-hash=668d59f6b7
... several seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE     LABELS
busybox-deployment-668d59f6b7-762nm   0/1     Terminating   0          6m49s   app=busybox,pod-template-hash=668d59f6b7
busybox-deployment-668d59f6b7-rm7lb   1/1     Terminating   0          6m49s   app=busybox,pod-template-hash=668d59f6b7

kubectl get pods --show-labels

No resources found.
terminationGracePeriodSeconds default value for Pods is 30 seconds. That is why termination takes so long.
We will do several Deployment demos and such a long delay every time is too long.
We will set terminationGracePeriodSeconds to 0 seconds : kill Pod immediately. VERY BAD during production but enables quick experiments during development.
2) terminationGracePeriodSeconds: 0
Note last line in Pod spec.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

      terminationGracePeriodSeconds: 0
Create the Deployment
kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
Show details while Deployment is in progress.
kubectl get pods --show-labels

NAME                                 READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-5d6889b44-7xrxh   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-8d56t   1/1     Running             0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-b7xbs   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-bng4z   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-cs5tx   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-fhrnf   1/1     Running             0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-jz985   1/1     Running             0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-mtmm2   0/1     Pending             0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-nv54h   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-v9rzg   0/1     ContainerCreating   0          3s    app=busybox,pod-template-hash=5d6889b44
Note several Pods already running ( since creation is so quick ). All other needed Pods are created simultaneously ( ContainerCreating )
Deployment complete in around 10 seconds.
kubectl get pods --show-labels

NAME                                 READY   STATUS    RESTARTS   AGE   LABELS
busybox-deployment-5d6889b44-7xrxh   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-8d56t   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-b7xbs   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-bng4z   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-cs5tx   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-fhrnf   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-jz985   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-mtmm2   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-nv54h   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
busybox-deployment-5d6889b44-v9rzg   1/1     Running   0          9s    app=busybox,pod-template-hash=5d6889b44
Now the test for terminationGracePeriodSeconds: 0
kubectl delete -f myDeployment.yaml --force --grace-period=0

deployment.apps "busybox-deployment" force deleted
One second later:
kubectl get pods --show-labels
No resources found.
All Pods for this Deployment got deleted immediately. 
Note that our previous example also had --force --grace-period=0 at the end.
That got ignored - each Pod still took 30 seconds to terminate.
Why? We have that --force flag on a delete command line for a Deployment. Deployments do not have a grace period so it got ignored.
The Deployment manages a ReplicaSet that manages our 10 Pods. The --force does not get passed from the Deployment on to the ReplicaSet. ReplicaSets do not have a --force option. 
So when ReplicaSets command the Pods to terminate, no --force flag is sent to them.
It would have been better if kubectl delete showed a warning when you add --force --grace-period=0 when deleting a Deployment.
... Warning : Deployments do not support --force --grace-period=0, flags ignored.
3) Updating the Image in a Pod
In this section we will update our Pod to use a different busybox image.
terminationGracePeriodSeconds is 10 seconds - otherwise things happen too fast to learn what is happening.
nano myDeployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

      terminationGracePeriodSeconds: 10
Create the Deployment .
kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
Repeatedly run command below until all 10 replicas are available.
kubectl get deploy

NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
busybox-deployment   10        10        10           10          1m
Currently our busybox is using version 1.30 - the latest version.
Let's assume we now need to run version 1.29.3.
We use the following command to update our Deployment to use busybox image 1.29.3
Note the --record at the end. This adds an annotation to the Deployment
kubectl set image deployment.v1.apps/busybox-deployment busybox=busybox:1.29.3 --record
Show ongoing status of Deployment ...
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-2vhm9   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-4sqlh   1/1     Running             0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-5jhwz   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-7lzdg   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-cxx7s   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ftxht   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-h8kxp   1/1     Running             0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hqd4s   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jwfvc   1/1     Terminating         0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-srfxd   1/1     Running             0          22s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-5cxs9    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-6v4z6    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7qmxt    0/1     Pending             0          1s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-cc7th    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-d6wt5    0/1     Pending             0          1s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dgzq4    0/1     ContainerCreating   0          4s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-fjk7t    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-jhrrj    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-pdwgm    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-thd5m    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-6b48b9cb8f is the prefix of our old Pods 
Several of them are terminating - making place for new Pods with busybox 1.29.3 
Several of them are still running - making it possible for our old Pods to still do work during the rollover update
busybox-deployment-8866dfbb4 is the prefix of our new Pods - busybox 1.29.3 
There is already several new Pods running 
There are 2 in ContainerCreating status - busy being created
2 Pods are in Pending status ... in a queue ... waiting to be created
... several seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-2vhm9   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-4sqlh   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-5jhwz   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-7lzdg   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-cxx7s   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ftxht   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-h8kxp   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hqd4s   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jwfvc   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-srfxd   1/1     Terminating         0          26s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-5cxs9    1/1     Running             0          6s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-6v4z6    1/1     Running             0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7qmxt    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-cc7th    1/1     Running             0          6s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-d6wt5    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dgzq4    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-fjk7t    1/1     Running             0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-jhrrj    1/1     Running             0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-pdwgm    1/1     Running             0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-thd5m    1/1     Running             0          9s    app=busybox,pod-template-hash=8866dfbb4
All old Pods are terminating.
All new Pods are running or being created: ContainerCreating 
... several seconds later ...
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-2vhm9   0/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-4sqlh   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-5jhwz   0/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-7lzdg   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-cxx7s   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ftxht   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-h8kxp   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hqd4s   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jwfvc   0/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-srfxd   1/1     Terminating   0          30s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-5cxs9    1/1     Running       0          10s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-6v4z6    1/1     Running       0          13s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7qmxt    1/1     Running       0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-cc7th    1/1     Running       0          10s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-d6wt5    1/1     Running       0          9s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dgzq4    1/1     Running       0          12s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-fjk7t    1/1     Running       0          13s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-jhrrj    1/1     Running       0          13s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-pdwgm    1/1     Running       0          13s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-thd5m    1/1     Running       0          13s   app=busybox,pod-template-hash=8866dfbb4
All new Pods are running.
If you do one more kubectl get pods --show-labels a few seconds later you will see all old Pods deleted.
Delete Pod.
kubectl delete -f myDeployment.yaml

deployment.apps "busybox-deployment" force deleted
4) maxUnavailable and maxSurge
Two critical settings control how a RollingUpdate works
• maxUnavailable https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable 
• maxSurge https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge 
Summary:
• maxUnavailable ... the maximum number of Pods that can be unavailable during the update process.
• maxSurge ... the maximum number of Pods that can be created IN ADDITION TO the desired number of Pods.
We are going to do several exercises so that you can become familiar with how this works.
These settings are easy to understand and they do work.
Unfortunately once a complex update is underway, it is difficult to PRECISELY see those settings are being observed 100% accurately. After these few exercises you will at least get the feeling it works as expected.
Both these fields are optional, since both have a default value of 25%.
We are going to set those fields to drastically different values so you can see the effect.
This exercise:
• maxSurge: 5 Pods can be created IN ADDITION TO the desired number of Pods.
• maxUnavailable: 5 Pods can be unavailable DURING the update process.
nano myDeployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 5
      maxUnavailable: 5
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

      terminationGracePeriodSeconds: 10
Create the Deployment
kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
Show Deployment in progress.
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-f992m   1/1     Running             0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-gt6wc   1/1     Running             0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ht94m   1/1     Terminating         0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jbz8j   1/1     Running             0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jjtqx   1/1     Terminating         0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-k8bcf   1/1     Terminating         0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-mwfnm   1/1     Terminating         0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-nqbml   1/1     Terminating         0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-rz5bj   1/1     Running             0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-v2fcf   1/1     Running             0          27s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-425zn    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-4hnmx    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7lmp7    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-b426t    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dbgvb    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-hl2m2    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-kr7dt    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qsvq8    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-vsmqp    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-wtfnc    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
• maxUnavailable: 5 Pods can be unavailable DURING the update process. WORKS AS EXPECTED.
Right at top. 5 Pods are right now Terminating = unavailable DURING the update process
• maxSurge: 5 Pods can be created IN ADDITION TO the desired number of Pods. WORKS AS EXPECTED.
All lines near bottom: 4 Pods are ContainerCreating = created IN ADDITION TO the desired number of Pods
In theory that should have been 5, but 4 is precise enough.
Several seconds later:
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-f992m   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-gt6wc   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ht94m   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jbz8j   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jjtqx   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-k8bcf   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-mwfnm   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-nqbml   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-rz5bj   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-v2fcf   1/1     Terminating         0          33s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-425zn    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-4hnmx    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7lmp7    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-b426t    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dbgvb    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-hl2m2    0/1     ContainerCreating   0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-kr7dt    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qsvq8    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-vsmqp    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-wtfnc    1/1     Running             0          8s    app=busybox,pod-template-hash=8866dfbb4
All old Pods are Terminating since we specified: maxUnavailable: 5 Pods can be unavailable DURING the update process. WORKS AS EXPECTED.
Only one Pod in ContainerCreating - all Pods at bottom are new ( only running for 8 seconds )
Several seconds later:
kubectl get pods --show-labels

NAME                                  READY   STATUS        RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-f992m   1/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-gt6wc   1/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ht94m   0/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-jbz8j   1/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-k8bcf   0/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-mwfnm   0/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-rz5bj   1/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-v2fcf   1/1     Terminating   0          39s   app=busybox,pod-template-hash=6b48b9cb8f

busybox-deployment-8866dfbb4-425zn    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-4hnmx    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7lmp7    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-b426t    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-dbgvb    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-hl2m2    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-kr7dt    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qsvq8    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-vsmqp    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-wtfnc    1/1     Running       0          14s   app=busybox,pod-template-hash=8866dfbb4
PS C:\k8>
ALL old Pods are Terminating They are taking their time since we specified: terminationGracePeriodSeconds: 10
All 10 Pods at bottom are new.
Deployment complete.
Delete Deployment .
kubectl delete -f myDeployment.yaml --force --grace-period=0

deployment.apps "busybox-deployment" force deleted
5) maxSurge: 5 maxUnavailable: 5
This exercise:
• maxSurge: 5 Pods can be created IN ADDITION TO the desired number of Pods.
• maxUnavailable: ONLY ONE Pods can be unavailable DURING the update process.
nano myDeployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 5
      maxUnavailable: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

      terminationGracePeriodSeconds: 10
Create the Deployment .
kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
Wait 10 seconds for this Deployment to complete.
kubectl get pods --show-labels

NAME                                  READY   STATUS    RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-54gkx   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-96w9b   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-cbpw4   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-h97jv   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hm8r8   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hp2jn   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-l5swd   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-nvcsz   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-q6ssv   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-qhtd7   1/1     Running   0          7s    app=busybox,pod-template-hash=6b48b9cb8f
PS C:\k8>
Submit command to change image the container uses to busybox:1.29.3
kubectl set image deployment.v1.apps/busybox-deployment busybox=busybox:1.29.3 --record

deployment.apps/busybox-deployment image updated
VERY QUICKLY afterwards run this:
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-54gkx   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-96w9b   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-cbpw4   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-h97jv   1/1     Terminating         0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hm8r8   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-hp2jn   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-l5swd   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-nvcsz   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-q6ssv   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-qhtd7   1/1     Running             0          35s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-6hbmn    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-9qmp8    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-b5b9h    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-bsbt2    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-cv8vz    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qrbmc    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
• maxUnavailable: ONLY ONE Pods can be unavailable DURING the update process.
Perfect: we see only ONE Terminating Pod.
• maxSurge: 5 Pods can be created IN ADDITION TO the desired number of Pods.
Since one Pod is terminating this 1 plus these 5 can be created: ContainerCreating = 6 Pods. 
No need to view further details - we saw those 2 settings work as expected.
Delete Deployment .
kubectl delete -f myDeployment.yaml --force --grace-period=0

deployment.apps "busybox-deployment" force deleted
6) maxSurge: 10 maxUnavailable: 10
This exercise:
• maxSurge: 10 Pods can be created IN ADDITION TO the desired number of Pods.
• maxUnavailable: ALL 10 Pods can be unavailable DURING the update process.
nano myDeployment.yaml 

apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
  labels:
    app: busybox
spec:
  replicas: 10
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 10
      maxUnavailable: 10
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']

      terminationGracePeriodSeconds: 10
Create the Deployment
kubectl create -f myDeployment.yaml

deployment.apps/busybox-deployment created
Immediately afterwards:
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-4x7dr   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-97284   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-bfrvv   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-fplhq   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ms8dh   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-prm86   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-rzknx   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-sptzx   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-vf4kd   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-wcwg4   1/1     Terminating         0          21s   app=busybox,pod-template-hash=6b48b9cb8f

busybox-deployment-8866dfbb4-2dzfc    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-2sdqj    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-6xr8w    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7s62s    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-d9zmg    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-nqn9k    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-pn2td    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qsq6d    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-rkmtk    0/1     Pending             0          2s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-shvxp    0/1     ContainerCreating   0          2s    app=busybox,pod-template-hash=8866dfbb4
• maxUnavailable: ALL 10 Pods can be unavailable DURING the update process.
ALL 10 old Pods are Terminating - as expected.
• maxSurge: 10 Pods can be created IN ADDITION TO the desired number of Pods.
Bottom 10 Pods are the new ones. Only 3 ContainerCreating ... NOT AS EXPECTED
I expected ALL 10 new Pods to be created at same time. ZERO Pending ones expected.
3 seconds later ... all new Pods still not ContainerCreating ... unexplained.
kubectl get pods --show-labels

NAME                                  READY   STATUS              RESTARTS   AGE   LABELS
busybox-deployment-6b48b9cb8f-4x7dr   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-97284   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-bfrvv   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-fplhq   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-ms8dh   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-prm86   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-rzknx   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-sptzx   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-vf4kd   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-6b48b9cb8f-wcwg4   1/1     Terminating         0          24s   app=busybox,pod-template-hash=6b48b9cb8f
busybox-deployment-8866dfbb4-2dzfc    0/1     Pending             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-2sdqj    0/1     Pending             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-6xr8w    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-7s62s    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-d9zmg    0/1     Pending             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-nqn9k    1/1     Running             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-pn2td    0/1     Pending             0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-qsq6d    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-rkmtk    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
busybox-deployment-8866dfbb4-shvxp    0/1     ContainerCreating   0          5s    app=busybox,pod-template-hash=8866dfbb4
Demo complete. maxUnavailable works 100% as expected.
Delete Deployment
kubectl delete -f myDeployment.yaml

deployment.apps "busybox-deployment" force deleted
Conclusion
Now that you have seen demos of different maxSurge and maxUnavailable settings, do this one on your own.
      maxSurge: 1
      maxUnavailable: 1
First consider what you learnt about maxSurge and maxUnavailable.
Then predict what you expect will happen. Then run the exercise to confirm your predictions.
Delete Deployment when done
kubectl delete -f myDeployment.yaml --force --grace-period=0

deployment.apps "busybox-deployment" force deleted
```

