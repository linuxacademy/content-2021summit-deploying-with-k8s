# Deploying and Scaling with Kubernetes

## Abstract
Kubernetes is a great tool for wrangling applications and implementing a variety of DevOps best practices. One of its strengths is its ability to manage deployments and application scaling. In this talk, we’ll dive into what it looks like to deploy and scale applications in a Kubernetes cluster. We’ll talk about what Kubernetes Deployments are, and their rolling update functionality. We’ll also talk about how you can easily scale up and down using Deployments.

## Links
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
- [Scaling](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

## Guide
Let's create a Pod.

```
vi my-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```
kubectl apply -f my-pod.yml

kubectl get pods
```

What if we want to make a change to the Pod? Let's try to deploy a new version of the code.

```
vi my-pod.yml
```

Change the `image:` to `nginx:1.19.10`.

```
kubectl apply -f my-pod.yml
```

It won't let us change the Pod. In k8s, you're supposed to replace old Pods with new Pods rather than making lots of edits to existing Pods.

So let's try a Deployment instead.

```
vi my-deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```
kubectl apply -f my-deployment.yml

kubectl get deployments

kubectl get pods
```

Now that we have several replicas of our app, we'll need to make sure users can access them without worrying about which Pod they're interacting with. Let's create a Service.

```
vi my-service.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-deployment
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
```

```
kubectl create -f my-service.yml

kubectl get services
```

Now we should be able to reach our app in a browser using port `30080` on any of our Kubernetes nodes.

```
http://<node IP address>:30080
```

Now let's try to deploy a new version.

```
vi my-deployment.yml
```

Change the `image:` to `nginx:1.19.10`.

```
kubectl apply -f my-deployment.yml

kubectl get pods
```

The Deployment manages the process of rolling out the new version, ensuring that the old Pod is not removed until the new one is fully ready. This is called a rolling update.

What if I want to scale up, and create more Pods?

```
vi my-deployment.yml
```

Change the `replicas:` to `5`.

```
kubectl apply -f my-deployment.yml

kubectl get deployments

kubectl get pods
```

You can do some very straightforward scaling using deployments.

Let's try and delete one of the Pods.

```
kubectl delete pod $pod_name

kubectl get pods
```

The Deployment will automatically create a new Pod to replace the one that was deleted. If I want to get rid of these pods, I would need to delete the deployment.

Let's do something a little more complex. Let's do a blue/green deployment with Kubernetes!

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 2
  selector:
    matchLabels:
      split: blue
  template:
    metadata:
      labels:
        split: blue
        app: bluegreen-test
    spec:
      containers:
      - name: app
        image: linuxacademycontent/summit-k8s:blue
        ports:
        - containerPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: green
spec:
  replicas: 2
  selector:
    matchLabels:
      split: green
  template:
    metadata:
      labels:
        split: green
        app: bluegreen-test
    spec:
      containers:
      - name: app
        image: linuxacademycontent/summit-k8s:green
        ports:
        - containerPort: 80

---

  apiVersion: v1
  kind: Service
  metadata:
    name: bluegreen-svc
  spec:
    selector:
      app: bluegreen-test
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30081
```

There's a lot you can do with Deployments in Kubernetes, but this should give you an idea of the basics!
