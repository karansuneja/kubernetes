
# kubectl commands

Usefull kubectl interactive commands

## POD
Create a Pod
```bash
  kubectl run nginx --image=nginx

```
Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```bash
  kubectl run nginx --image=nginx --dry-run=client -o yaml

```
## Deployment
Create a deployment
```bash
kubectl create deployment --image=nginx nginx
```


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment with 4 Replicas
```bash
kubectl create deployment nginx --image=nginx --replicas=4
```


To Scale a deployment
```bash
kubectl scale deployment nginx --replicas=4
```

To save a definition in yaml before creating it.
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

 
## Service

Creating a service
```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
(This will automatically use the pod's labels as selectors)

Or
```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
```
(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or
```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

## Scheduling in Kubernetes

nodeName - Manually scheduling a pod to a node

```bash
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeName: my-node
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80

```

Labels And selectors

```bash
kubectl get pods --selector app=App1
```
```bash
kubectl get pods -l env=prod,app=nginx
```

In the file below, the selector under spec and the label under template are the same. This ensures that the Deployment manages the Pods correctly by grouping them together.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: front-end
  template:
    metadata:
      labels:
        tier: front-end
    spec:
      containers:
        - name: nginx
          image: nginx
```

## Taints and Tolerations

Taints

taints are applied on node.
```bash
kuectl taint nodes node-name key=value:{taint-effect}
```

values for taint-effect can be:
- NoSchedule -> pods will not be scheduled on node.
- PreferNoSchedule -> system will try to not schedule pod on the node.
- NoExecute -> new pods will not schedule and existing nodes will get evicted to another node.

Tolerations

tolerations are applied on pods

tolerations are same value as taints applied , key as key value , operator as operator used, value as value, effect as effect set for taint node.
```bash
spec:
 containers:
 - name: nginx-container
   image: nginx
 tolerations:
 - key: "app"
   operator: "Equal"
   value: "blue"
   effect: "NoSchedule"
```

```bash
kubectl describe node {nodeName} | grep Taints
```

To remove a tain from node.  just append - at end
```bash
kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

## Documentation

[kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

[kubectl-convention](https://kubernetes.io/docs/reference/kubectl/conventions/)

