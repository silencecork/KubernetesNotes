Deployment
===

<pre>
apiVersion: apps/v1
<b>kind: Deployment</b>
metadata:
  name: deploytest
  namespace: dev
spec:
  <b>replica: 3</b>
  <b>selector:</b>
    matchLabels:
      name: test-pod
  template:
    metadata:
      labels:
        name: test-pod
    spec:
      containers:
      - name: test-pod-container
        image: nginx
</pre>

Commands
===

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod-container
    image: nginx
    <b>command:
    - sleep
    - "2000"</b>
</pre>
or
<pre>
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod-container
    image: nginx
    <b>command: ["sleep"]
    args: ["--time", "1000"]</b>
</pre>

Config Map
===

Define ConfigMap
---

<pre>
apiVersion: v1
kind: ConfigMap
metadata:
  name: <font color='green'>my-config-map</font>
<b>data:
  MY_MAP_KEY_1: VALUE_1
  MY_MAP_KEY_2: VALUE_2</b>
</pre>

Environment Variable define Directly
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    env:
    - name: MY_KEY_1
      value: VALUE_1
</pre>

Use ConfigMap
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    <b>envFrom:
      - configMapRef:
          name: <font color='green'>my-config-map</font></b>
</pre>

Use ConfigMap and search by key 
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    env:
    - name: MY_KEY_1
      <b>valueFrom:
        configMapKeyRef:
          name: <font color='green'>my-config-map</font>
          key: MY_MAP_KEY_1</b>
</pre>

Mount ConfigMap as Volume
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>volumes:</b>
  - name: my-configmap-volume
    <b>configMap:
      name: <font color='green'>my-config-map</font></b>
  containers:
  - name: my-pod-container
    image: nginx
    <b>volumeMounts:
    - name: my-volume
      mountPath: "/etc/config"</b>
</pre>

Secret
===

Define Secret
---

<pre>
apiVersion: v1
<b>kind: Secret</b>
metadata:
  name: my-secret
data: 
  SECRET_1: <b>base64(</b>VALUE_1<b>)</b>
</pre>

Use command for base64 encode
---

##### Encode
```console
$ echo -n 'xxxx' | base64
```
##### Decode
```console
$ echo -n 'eHh4eAo=' | base64 --decode
```

Use in Pod as whole env variable
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    <b>envFrom:
    - secretRef:
        name: my-secret</b>
</pre>

Use in Pod as value reference of env variable
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    env:
    - name: MY_ENV_1
      <b>valueFrom:
        secretKeyRef:
          name: my-secret
          key: SECRET_1</b>
</pre>

Mount Secret as Volume
---

<pre>
apVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>volumes:
  - name: secret-volume
    secret:
      secretName: my-secret</b>
  containers:
  - name: my-pod-container
    image: nginx
    <b>volumeMounts:
    - name: mounted-secret
      mountPath: "/etc/foo"
      readOnly: false</b>
</pre>

Security Context
===

Let Pod run as specific level of User
<br>Pod can set user and container can overwrite it
<br>use **capabilities** to add permission
<br>**only container** can use capabilities

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>securityContext:
    runAsUser: 1000</b>
  containers:
  - name: my-pod-container
    image: nginx
    <b>securityContext:
      runAsUser: 1001
      capabilities:
        add: ["SYS_TIME"]</b>
</pre>

Service Account
===

Create sevice account
---

```console
$ kubectl create serviceaccount [NAME]
```

Check tokens
---

```console
$ kubectl describe serviceaccount [NAME]
```
<br> above commands can find "Tokens" attribute with a name, here we called it [TOKENS_NAME] and <br>
```console
$ kubectl describe secrets [TOKENS_NAME]
```
<br>You can find real token

Use service account in Deployment
---

<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replica: 1
  selector:
    matchLabels: 
      name=my-deployment
  template:
    <b>serviceAccount: [NAME]</b>
    metadata:
      labels:
        name=my-deployment
    spec:
      containers:
      - name: my-deployment-container
        image: nginx
</pre>

Resource
===

#### default is 0.5 CPU and 256 Mi

Define Resource in Pod
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    ports:
    - containerPort: 8080
    <b>resources:
      request:
        memory: "1Gi"
        cpu: 2
      limit:
        memory: "2Gi"
        cpu: 4</b>
</pre>

Taints and Tolerations
===

- Taint on Node
- Tolerant on Pod

Add taint to node
---

```console 
$ kubectl taint node [node name] [taint name]:[effect] 
```
- [taint name]: could be a <b>[traint key]=[traint value]</b> or a name like node-role.kubernetes.io/master
- [effect]: NoSchedule / NoExecute / PreferNoSchedule

Remove taint from node
---

```console
$ kubectl taint node [node name] [taint name]-
```
#### Remember! add minus(-) at last...

Add tolerations to Pod
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>tolerations:
  - key: "[taint key]"
    value: "[taint value]"
    operator: "Equal"
    effect: "[taint effect]"</b>
  containers:
  - name: my-pod-container
    image: nginx
</pre>
#### Taint only can prevent pod executed on specific Node, but it can NOT let Pod executed on specific Node!

Node Selector, Node Affinity
===

Why we need affinity? 
---
#### Node Selector not support OR or NOT options

Add Label to Node
---

```console
$ kubectl label node <NODE name> [key]=[value]
```
e.g. 
```console
$ kubectl label node node01 size=Large
```

Create Node Selector on Pod
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
  <b>nodeSelector:
    [key]: [value] </b>
</pre>

Use Node Affinity
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
  <b>affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - key: [key]
          operator: In
          values:
          - [value1]
          - [value2] </b>
</pre>
#### operator got In, NotIn, Exists ...


#### If you want make sure pod executed in specific node, and other pod can not execute in

1. taint node
2. set Tolerations on Pod
3. set label to node
4. set node affinity on Pod

Multicontainer Pod
===

Empty...

Readiness & Liveness
===

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-pod-container
    image: nginx
    ports:
    - containerPort: 8080
    <b>readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 1
      failureThreshold: 10</b>

    <b>livenessProbe:
      httpGet:
        path: /
        port:
      initialDelaySeconds: 10
      periodSeconds: 1
      failureThreshold: 10</b>
</pre>

Logs
===

```console
$ kubectl logs -f [pod name]
```
<br>if more than one container in Pod<br>
```console
$ kubectl logs -f [pod name] [container name] 
```

Selector
===

```console
$ kubectl get [resource] \
--selector label_key=label_value \
--selector label_key=label_value
```

Rolling Update & Rollback
===

Strategy - Recreate
---

<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replica: 4
  selector:
    matchLabels:
      name: frontend
  <b>
    strategy:
      type: Recreate</b>
</pre>

Strategy - RollingUpdate (Default)
---

<pre><b>
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 20%
      maxunavailable: 10%</b>
  template:
    metadat:
      name: my-pod
      labels: 
        name: frontend
    spec:
      containers:
      - name: my-pod-container
        image: nginx
        ports:
        - containerPort: 8080
</pre>

Begin Rollout
---

```console
$ kubectl create -f myapp-deployment.yml
```

Check Rollout status
---

```console
$ kubectl rollout status deployment/myapp-deployment
```

Check Rollout history
---

```console
$ kubectl rollout history deployment/myapp-deployment
```

Rollout new version
---

```console
$ kubectl apply -f myapp-deployment-new.yml
```
or
```console
$ kubectl set image deployment/myapp-deployment [container name]=[image name]
```

Rollback
---

```console
$ kubectl rollout undo deployment/myapp-deployment
```

Jobs & CronJob
===

Jobs
---

<pre>
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:<b>
  completions: 3
  parallelism: 3</b>
  template:
    spec:
      container:
      - name: my-job-container
        image: nginx
      <b>restartPolicy: Never</b>
</pre>

CronJob 
---

##### CronJob is Cron schedule define and include job spec
<pre>
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cron-job
spec:
  <b>schedule: "* * * * *"</b>
  jobTemplate:
    ... Job define ....
</pre>
<pre>
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cron-job
spce:
  <b>schedule: "0 12 * * *"</b>
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
          - name: my-cronjob-container
            image: nginx
          restartPolicy: Never
</pre>        

Service
===

NodePort
---

<pre>
apiVersion: v1
<b>kind: Service</b>
metadata:
  name: my-service
spec:
  <b>type: NodePort
  ports:
  - targetPort: 8080
    port: 80
    nodePort: 30008</b>
  selector:
    app: my-app
    type: frontend
</pre>

ClusterIP (Default)
---

<pre>
apiVersion: v1
<b>kind: Service</b>
metadata:
  name: my-service
spec:
  <b>type: ClusterIP
  ports:
  - port: 80
    targetPort: 80</b>
  selector:
    app: my-app
    type: frontend
</pre>

Network Policy
===

```console
$ kubectl get networkpolicy
```
##### K8S default not support Network Policy

Ingress
---
##### The network traffic to a Pod

<pre>
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-ingress-policy
spec:
  <b>podSelector:</b>
    matchLabels:
      name: my-pod
  <b>policyTypes:
  - Ingress</b>

  <b>ingress:
  - from:
    - podSelector:
        matchLabels:
          name: other-pod-in
    ports:
    - protocol: TCP
      port: 8080</b>
</pre>

Egress
---

##### Network traffic from Pod to somewhere
<pre>
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-egress-policy
spec:
  podSelector:
    matchLabels:
      name: my-pod
  <b>policyTypes:
  - Egress</b>

  <b>egress:
  - to:</b>
    - podSelector:
        matchLabels:
          name: to-other-pod
    ports:
    - port: 3306
      protocol: TCP
</pre>

Volume & PersistentVolume & PersistentVolumeClaim
===

```console
$ kubectl get pv
```
```console
$ kubectl get pvc
```

Define a Volume
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>volumes:
  - name: <font color='red'>my-volume</font>
    hostPath:
      path: [Host Path]
      type: Directory</b>
  containers:
  - name: my-pod-container
    image: nginx
    <b>volumeMounts:
    - mountPath: [container path]
      name: <font color='red'>my-volume</font></b>
</pre>

Define a PersistentVolume
---

<pre>
apiVersion: v1
<b>kind: PersistentVolume</b>
metadata:
  name: my-pv
<b>spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  hostPath:
    path: [Host path]
  peristentVolumeReclaimPolicy: Retain</b>
</pre>

Define a PersistentVolume
---

#### on host
<pre>
apiVersion: v1
<b>kind: PersistentVolume</b>
metadata:
  name: my-pv
<b>spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  hostPath:
    path: [Host path]
  peristentVolumeReclaimPolicy: Retain</b>
</pre>
#### on NFS
<pre>
apiVersion: v1
<b>kind: PersistentVolume</b>
metadata:
  name: my-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  <b>nfs:
    path: [NFS path]
    server: [NFS server]</b>
  peristentVolumeReclaimPolicy: Retain
</pre>

Use in Pod
---

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  <b>volumes:</b>
  - name: <font color='blue'>my-volume-name</font>
    hostPath:
      path: [Host path]
      type: Directory
  - name: <font color='yellow'>my-pvc-name</font>
    <b>persistentVolumeClaim:</b>
      claimName: my-claim-name

  containers:
  - name: my-pod-container
    image: nginx
    <b>volumeMounts:
    - mountPath: [containers path]
      name: <font color='blue'>my-volume-name</font>
    - mountPath: [containers path]
      name: <font color='yellow'>my-pvc-name</font></b>
</pre>

Kubectl Commands
===

Nodes
---

```console
$ kubectl get nodes
$ kubectl get node [NAME] -o wide (or yaml or json)
$ kubectl get nodes --show-labels
```

Label
---

#### set Label to a node
```console
$ kubectl label node [NAME] key=value
```
#### remove Label from a node
```console
$ kubectl label node [NAME] key-
```

Role
---

#### set Role to a node
```console
$ kubectl label node [NAME] \ 
node-role.kubernetes.io/[role]=\
node-role.kubernetes.io/master=
```

Context
---

#### get all Contexts
```console
$ kubectl config get-contexts
```
#### change Context
```console
$ kubectl config set current-context [context name]
```
```console
$ kubectl config use-context [context name]
```
#### delete Context
```console
$ kubectl config delete-context [NAME]
```

Pod
---

#### get Pods
```console
$ kubectl get pods (--all-namespaces) (--namespace=[space name])
$ kubectl get po
```
#### execute command in Pod
```console
$ kubectl exec [pod name] -it sh (or other command)
```
#### execute command in container of Pod
```console
$ kubectl exec [pod name] -c [container name] -it sh (or other command)
```
#### delete a Pod
```console
$ kubectl delete pod [NAME]
$ kubectl delete po [NAME]
```

Namespace
---

#### get all Namespaces
```console
$ kubectl get namespace
$ kubectl get ns
```
#### create a Namespace
```console
$ kubectl create namespace [NAME]
$ kubectl create ns [NAME]
```
#### delete a Namespace
```console
$ kubectl delete namespace [NAME]
$ kubectl delete ns [NAME]
```

Deployment
---

#### get all Deployments
```console
$ kubectl get deployment
$ kubectl get deploy
```
#### delete a Deployment
```console
$ kubectl delete deployment [NAME]
$ kubectl delete deploy [NAME]
```
#### scale a Deployment
```console
$ kubectl scale --current-replicas=[num] --replicas=[num] deployment/[NAME]
```
#### rollout
##### rollout a Deployment
```console
$ kubectl set image deployment/[NAME] [container name]=[image name]:[new version]
```
##### rollout status
```console
$ kubectl rollout status deployment/[NAME]
```
##### rollout history
```console
$ kubectl rollout history deployment/[NAME]
```
##### check difference between rollout history
```console
$ kubectl rollout history deployment/[NAME] --revision=[num]
```
##### undo rollout
```console
$ kubectl rollout undo deployment/[NAME]
```
##### undo rollout to specific version
```console
$ kubectl rollout undo deployment/[NAME] --to-revision=[num]
```

ReplicaSet
---

#### get all ReplicaSet
```console
$ kubectl get replicaset
$ kubectl get rs
```

Create Pod YAML Tips
===

Create an NGINX Pod
---

```shell
$ kubectl run --generator=run-pod/v1 nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
---


```shell
$ kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
```

Create a deployment
---

```shell
$ kubectl run --generator=deployment/v1beta1 nginx --image=nginx
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
---

```shell
$ kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml
```

##### with 4 replicas
```shell
$ kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
```

Save it to a file
---

```shell
$ kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
```

Output Yaml from running Pod
---

```shell
$ kubectl get po <name> -o yaml --export > file.yml
```