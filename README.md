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
  name: app-in-special-node
  labels:
    app: app-in-special-node
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-in-special-node
  template:
    metadata:
      labels:
        app: app-in-special-node
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

You can deploy it by running  `kubectl apply -f k8s.yaml`.

## nodeSelectors and okteto

Once your application is up and running, you can use the [okteto CLI](https://github.com/okteto/okteto) to develop directly on your cluster, with the exact same configuration that you use for your application, `nodeSelector` included. 

As long as the name in your `okteto.yaml` is the same as the name of the existing deployment, `okteto` will automatically use that deployment. Notice how in the example below we use `app-in-special-node` as the name, which is the same name that we used in your `deployment`, in the first part of this doc.

```
name: app-in-special-node
image: okteto/golang:1
command: bash
sync:
- .:/usr/src/app
forward:
- 8080:8080
```

Without explaining every line of the above manifest, let’s highlight and explain a few of them:
- `name`:  the name of the existing Kubernetes deployment we want to use.
- `image`: the image used by the development container.
- `command`: the start command of the development container.

> You can get more details about the Okteto manifest at our [documentation site](https://okteto.com/docs/reference/manifest/index.html).

All that's left is for you to run `okteto up` for okteto to deploy your development environment. And in this case, it won't just be in Kubernetes, but it will also be running in the same nodes that your application uses, the ones with the `big-worker-node=true` and `gpu=nvidia` labels.

