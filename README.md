# Deploy your Kubernetes Application to Specific Nodes

When deploying an application, the Kubernetes scheduler will typically deploy your application in the last busy node. But there are times where we might have more than one kind of node in our cluster. It could be that we have some nodes with more RAM, or maybe that we have nodes with specialized GPUs.

Imagine that you're building a machine learning model. In that case, you want Kubernetes to deploy your application to a very specific set of nodes (the ones with GPU), not to the least busy one. Enter **nodeSelectors*. 

## nodeSelectors

`nodeSelectors` is the simplest recommended form of node selection constraint. It's a field of `PodSpec` that specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node *must have each of the indicated key-value pairs as labels*.

For example, take a look at this Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: special-node
  labels:
    app: special-node
spec:
  replicas: 2
  selector:
    matchLabels:
      app: special-node
  template:
    metadata:
      labels:
        app: special-node
    spec:
      containers:
      - name: api
        image: bitnami/nginx
        ports:
        - containerPort: 8080
      nodeSelector:
        big-worker-node: "true"
        gpu: "nvidia"
```

This definition is telling Kubernetes to only place the pod in Nodes that have the `big-worker-node=true` and `gpu=nvidia` labels. Any node that has at least those labels will qualify to run my pods. 

## nodeSelectors and okteto

Once your application is up and running, you can use the okteto CLI to develop directly on it. The `okteto` CLI automatically inherits any configuration from the deployment, which in this case,  means that it will use the same `nodeSelector` configuration. This will result in your development container running in the same nodes that your application, the ones with the `big-worker-node=true` and `gpu=nvidia` labels.