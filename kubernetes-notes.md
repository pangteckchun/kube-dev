## Self-Mastery 101: Kubernetes
This contains all self-mastery notes on Kubernetes, starting from basics, security, helm to HA concepts.  

---

## Kubernetes Basics ##
1. What is Kubernetes (K8s)?  
*"It is an open-source system for automating deployment, scaling, and management of containerized applications"*

1. Written in Go programming language.

1. Communications to/fro and within K8s cluster are entirely API-driven. Cluster config information is stored in JSON format inside of `etcd` (database), but (mostly) written in YAML format. The K8s agents will conver the YAML into JSON format before use.

1. One challenge of adopting K8s for applications is the need for the applications to be rewritten or written so that their nature is truly transient (see 12 factor app principles @ https://12factor.net/)

1. Other than K8s, there are also other competing container orchestration and management solutions:
- Docker Swarm - provided by Docker Inc  
- Apache Mesos - a datacenter scheduler and using its own `Marathon` framework to orchestrate containers
- Nomad - provided by HashiCorp and schedules containers as tasks
- Rancher - orchestrator-agnostic system and supports Mesos, Swarm and K8s through its single pane of glass. Also touted as the enterprise Kubernetes management platform.

### Kubernetes Architecture ###
![Kubernetes Architecture](./img/kubernetes-architecture-overview.png)  

1. Central manager is called the `master` and it controls the `kube-apiserver`, `kube-scheduler`, various controllers and `etcd` storage system (cluster state, container settings and networking config). For now, only Linux nodes can be master on a cluster.

1. `kube-apiserver` exposes API for others to speak to K8s. This is also what you communicate with using native K8s client `kubectl` or using curl commands.

1. `Kubeadm` is a tool built to provide **kubeadm init** and **kubeadm join** as best-practice "fast paths" for creating K8s clusters. `Kubeadm` performs the actions necessary to get a minimum viable cluster up and running.

1. `kube-scheduler` takes requests for running containers (i.e. `kubectl` --> `kube-apiserver`) and finds a suitabl node to run that container.

1. Every `node` running a container would have `kubelet` and `kube-proxy` processes.

1. `kubelet` receives the request to run the containers, watches over them locally, manages the resources needed to keep the running. Interacts with the local container engine, Docker by default.

1. `kube-proxy` creates and manages networking rules to expose the container on the network.

1. A `Pod` is larger object and consists of one or more containers which share IP address, access to storage and namespace. Usually 1 container in a `Pod` runs an application, while other containers support the primary application. A `Pod` runs on a `node`.

1. Namespaces are used to segregate resources and objects from each other and important for multi-tenancy.

1. `Label` is an arbituary string which becomes part of K8s objects meta-data and can used to check the state or change state of objects without having to know their individual names or UIDs, which is difficult to know given we can manage thousands of `Pods` across hundreds of `nodes`.

1. Orchestration is managed through operators known as **controllers**. They all talk through the `kube-apiserver` to interrogate and modify and object states. All controllers are compiled into `kube-controller-manager` and custom ones are added using `CRDs` (custom resource definitions).

1. The default and most feature-riched operator is `Deployment` which manages `ReplicaSets` operator, which creates or terminates a `Pod` using **podSpec**. The **podSpec** is sent to a `kubelet` --> container engine (e.g. Docker) to spawn or terminate containers to match the number in the `ReplicaSet`.

1. The `Service` operator uses `labels` to request existing IP addresses and endpoints and manage the network connectivity between pods, namespaces and outside the cluster.

1. `Jobs` and `CronJobs` are for recurring tasks on the cluster.

---
