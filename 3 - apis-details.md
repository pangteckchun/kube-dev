## Kubernetes API & Access
This contains all concepts related to K8s REST-based architecture, API flow, API versions, Pod templates, annnotations, namespaces handling.  

---

### API Concepts ###
1. Starting with v1.16 deprecated objects are no longer honored by the `kube-apiserver`.

1. The entire Kubernetes API uses a Swagger specification but evolving towards OpenAPI initiative.

1. `kubectl` makes REST API calls on your behalf, responding to typical HTTP verbs (GET, POST, DELETE). You can also make calls externally, using curl or other program.

1. The default serialization for REST API calls must be JSON. While we may work with files in a YAML format, they are converted to and from JSON during invocations.

1. Kubernetes implements an alternative Protobuf based serialization format that is primarily intended for intra-cluster communication.

1. Kubernetes stores the serialized state of objects by writing them into etcd. By default, Kubernetes returns objects serialized to JSON with content type `application/json`.
`
1. The `kube-apiserver` handles the conversion between API versions transparently: all the different versions are actually representations of the same persisted data. `Kube-apiserver` may serve the same underlying data through multiple API versions.

1. Most Kubernetes API resource types are **objects**, such as a Pod, a cluster, namespaces, configMaps & events. All resource types are scoped by the cluster (`/apis/GROUP/VERSION/*`) or to a namespace (`/apis/GROUP/VERSION/namespaces/NAMESPACE/*`).

1. Every Kubernetes object has a **resourceVersion** field representing the version of that resource as stored in the underlying database. This allows a client to fetch the current state and then watch for changes without missing any updates.

1. **Annotations** are used in the metadata to be included with an object that may be helpful outside of the Kubernetes object interaction. They are key to value maps and more human readable than labels. For example, to annotate Pods withing a namespace: `kubectl annotate pods --all description='Production Pods' -n prod`.

1. **Namespaces** are means to keep resources distinct. Every API call includes a namespace, using default if not otherwise declared: `https://10.128.0.3:6443/api/v1/namespaces/default/pods`.

1. There are four namepaces when a cluster is created:
- **default** - Default where all resources will be if not set otherwise.
- **kube-node-lease** - Where worker node lease infomation are kept.
- **kube-pubic** - readale by all, even those authenticated.
- **kube-system** - For infrastructure Pods.

13. API versions are tagged as follows:
- **Alpha** - maybe buggy and is disabled by default and features change and disappear any time.
- **Beta** - More well tested code, enabled by default. Changes are tested for backwards compatibility between versions. Still expect bugs and issues.
- **Stable** - Denoted by an integer, preceded by a letter **v**.

14. Make a dry-run request by setting the **dryRun** query parameter. This parameter is a string, working as an enum, and the only accepted values are: **All** - same behaviour and result wil return to use except no database is updated. Leave the field empty is the default behaviour of no dry run.

1. Dry run requests need to be authorized just like non dry-run requests.

---

### More API objects ###
1. Always look at `v1` for stable release version for usage.

1. From v.16 onwards, deprecated apis will respond with errors.

1. There 8 groups of `v1` apis. We can query the apis using curl and we will need to obtain bearer token in order for us to have the authorisation to query the detais (see lab exercise on the know-how).

1. Note that these controller objects are used whenever we execute `kubectl create` command:
- **Deployment** - this is 1st created when command is run and it also creates the **ReplicaSet** object on our behalf.
- **ReplicaSet** - Creates the Pod when called upon by **Deployment** controller, and it manages the individual Pod lifecycle.
- **Pod** - see subsection above.
Therefore **Deployment** ---*create and manage*---> **ReplicaSet** ---*create and manage*---> **Pod**.

5. **DaemonSet** - for deploying specific type of Pod on a node, e.g. a logging application to all nodes. Ensures a singe Pod of the same capability runs on evert node in the cluster. Typically used for logging, metrics and security.

1. **StatefuSet** - Different from **Deployment** as it considers each Pod as unique and provides deployment ordering. Uses the same `PodSpec` as **Deployment** but assigned stable storage, network ID that remains with the node.

1. **Horizontal Pod Autoscaler** (HPA) - auto scales replication controllers, ReplicaSets & Deployment object balancing 50% CPU usage by default and starts to scale. It uses Metrics Server API to request for info and adjust replicas to match.

1. **Cluster Autoscaler** (CA) - auto scale nodes in cluster; also if Pod failure or low utilization for >=10 mins, remove node. This helps to minimise expenses from unused nodes.

1. **Vertical Autoscaler** (VA) - still in project state and NOT stable release yet. Auto adjusts the CPU & memory of individual Pods.

1. **Jobs** - Part of the `batch` api group.
- It sets a number of Pods to run to completion to fulfill a task. It tracks all Pods complete successfully before job is marked completed.
- Deleting a job will clean up the Pods it created.
- When jobs are completed, the Pods are not deleted automatically so you can view the logs for your own diagnosis. The job object also remains so you can check its status. It is the client's responsibility to delete the old jobs (which will purge the Pods) themselves.
- Job failures also require manual intervention to diagnose and resolve.
- Use a `JobSpec` to create a job object. Within the specs, attribute: *RestartPolicy* take either value[Never | OnFailure]. *RestartPolicy* applies to the Pods created and not the job object itself.
- If *RestartPolicy* == OnFailure, the Pod stays on the node, but the container is re-run on the same node.
- If *RestartPolicy* == Never, a new Pod will be started by the job controller.

11. There are 3 types of tasks that are suitable to run as jobs:
- Non parallel tasks --> which runs on 1 Pod only;
- Parallel tasks with fixed completion count;
- Parallel tasks with work queue which uses *spec.parallelism*. Pods coordinate amongst themselves to decide what each will work on.

1. *spec.backoffLimit* specifies the number of retries before a job is considered failed. The default is 6.

1. **RBAC** - 4 types of resources for us to define permissions:
- ClusterRole, Role, ClusterRoleBinding, RoleBinding.
- They each contain rules that represent a set of permissions. Permissions are purely additive (there are no "deny" rules).
We use them to define roles for a cluster & associate users to these roles.
- More about Kubernetes security, which incudes RBAC, linked [here](./kubernetes-security.md)