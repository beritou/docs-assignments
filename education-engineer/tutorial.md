# Deploy Your First Kubernetes App

Kubernetes can feel complex at first, but learning by doing makes it easier to understand.

In this tutorial, you will:

1. Create a local Kubernetes cluster using [Kind](https://kind.sigs.k8s.io/).
2. Deploy a simple web application.
3. View the application in your browser.

By the end, you’ll have hands-on experience deploying and managing applications in Kubernetes.

# Prerequisites

Make sure you have these tools installed before you begin:

- Docker  
  - [Install on macOS](https://docs.docker.com/desktop/setup/install/mac-install/)  
  - [Install on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
  - [Install on Linux](https://docs.docker.com/desktop/setup/install/linux/)  

- kubectl
  - [Install on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)  
  - [Install on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
  - [Install on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)  

---

# Install Kind

[Kind](https://kind.sigs.k8s.io/) lets you operate Kubernetes clusters locally inside Docker containers. You’ll use it to create your first cluster and deploy an application.

If Kind is already installed, you can skip this section.

## Install Kind

Install Kind by following the instructions in [the official quick start guide](https://kind.sigs.k8s.io/docs/user/quick-start#installation).

## Verify the Installation

Check that Kind is installed and working by printing its version:

```shell
kind --version
```

Expected output:

```
kind version 0.30.0
```

If the version number appears, Kind is ready to use.

# Start Docker

If Docker is already started, you can skip this section.

Start Docker Desktop.

Confirm that Docker is active by checking its status:

```shell
docker info
```

If Docker is started, the output shows system information such as version numbers and storage details. Your exact system information may be slightly different.

```shell
Client: Docker Engine - Community
 Version:    28.4.0
 Context:    desktop-linux
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
  compose: Docker Compose (Docker Inc.)
  scout: Docker Scout (Docker Inc.)
...

Server:
 Containers: 3
 Images: 2
 Server Version: 28.4.0
 Storage Driver: overlayfs
 Operating System: Docker Desktop
 Architecture: aarch64
 CPUs: 8
 Total Memory: 7.654GiB
...
```

Docker is now ready. Next, you’ll use Kind to create your first Kubernetes cluster inside Docker.

# Create Your First Kubernetes Cluster

Every Kubernetes environment starts with a cluster, which is a set of nodes managed by a control plane. Kind simplifies this by creating everything you need inside Docker containers on your local machine.

## Create the Cluster

You will now create a single-node cluster that behaves like a real Kubernetes setup but works locally.

Execute the following command to create the cluster:

```shell
kind create cluster
```

You should see output as Kind downloads dependencies and configures your cluster:

```shell
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Kind just created a complete Kubernetes cluster inside Docker. This local cluster is ready for you to deploy applications.

Before deploying anything, next, you'll confirm that Kubernetes is up and `kubectl` can reach it.

## Verify Kubernetes Connectivity

Confirm that your local `kubectl` command can reach the Kind cluster using the CLI command `kubectl cluster-info --context kind-kind`:

```shell
kubectl cluster-info --context kind-kind
```

You should see output showing the control plane and CoreDNS addresses:

```shell
Kubernetes control plane is running at https://127.0.0.1:55037
CoreDNS is running at https://127.0.0.1:55037/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

This confirms your Kind cluster is healthy and `kubectl` can communicate with it. Now it’s time to deploy your first application.

# Deploy an Application

In your current working directory, create a file named `app.yaml`.

Insert the following configuration into `app.yaml`:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

This configuration defines two key Kubernetes resources:

- **Deployment**: Tells Kubernetes which container image to use and how many application instances are needed.
- **Service**: Provides a way to reach the application inside the cluster using a name and port number.

Together, these resources describe both what to execute and how to access it.

Deploy the application using the `kubectl apply -f app.yaml` command:

```shell
kubectl apply -f app.yaml
```

You should see output confirming that the deployment and service were created:

```shell
deployment.apps/web created
service/web created
```

The application is now active and reachable inside the cluster.

Next, you’ll make it accessible from your local machine.

# Expose Application to the Local Network

Retrieve the name of the active Kubernetes pod and store it in a variable named `PODNAME` using the following `kubectl` command:

```shell
PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web) && echo "$PODNAME"
```

Verify the output shows the name of the active Kubernetes pod. The exact name will be slightly different, but it will look something like this:

```shell
web-769bbccc48-p4qfg
```

Use the pod name to expose the application to the local network with port forwarding by executing the following command:

```shell
kubectl port-forward $PODNAME 8080:8080
```

Verify the output shows the ports being forwarded:

```shell
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

The port forwarding command connects your local port 8080 to the app active in the cluster. Traffic from your browser will now reach the containerized application inside Kubernetes.

# Manually Test the Application

Now that the port is open, you can view your deployed application.

In a web browser, navigate to [http://localhost:8080/](http://localhost:8080/).

Verify your web browser displays the Hello World welcome page:

```
Hello, world!
Version: 1.0.0
Hostname: web-769bbccc48-p4qfg
```

You successfully deployed and accessed a web application managed by Kubernetes. Kind, Docker, and Kubernetes are now working together as one system.

# Cleanup

It is a good practice to remove resources when you are finished working so your system stays clean and ready for future work.
 
If port forwarding is still active, stop it first. Return to the terminal where you started the command and press **Ctrl + C** to end it.

```shell
Handling connection for 8080
^C
```

Once port forwarding has stopped, delete your local Kind cluster to remove all Kubernetes resources and free up system memory:

```shell
kind delete cluster
```

You should see confirmation output similar to the following:

```shell
Deleting cluster "kind" ...
Deleted nodes: ["kind-control-plane"]
```

Finally, remove the `app.yaml` file from your working directory to keep things tidy:

```shell
rm app.yaml
```

Your local Kubernetes environment is now cleaned up and ready for whatever you want to do next.

# Next Steps

Now that you have experienced the full deployment cycle in Kubernetes, here are a few ways to continue learning:

- Experiment with editing some of the configuration in `app.yaml`, such as the number of replicas.
- Explore the [Kubernetes Concepts](https://kubernetes.io/docs/concepts/) section of the official docs to understand objects like Pods, Services, and Deployments in more depth.
- Move to the cloud with a managed Kubernetes service like [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) or [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/).
- Go even further with [Spectro Cloud](https://www.spectrocloud.com/), which simplifies multi-cluster Kubernetes management.
