
## How to create a kind cluster

To create a cluster with `kind` We need to write a `.yaml` file with cluster specifications. Save the following steps in a file named  `kind.yaml`.

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

To create a cluster use:

```bsh
kind create cluster --name modern-stack --config kind.yaml
```

You can now check your cluster with:

```bsh
kubectl cluster-info --context kind-modern-stack

$ Kubernetes control plane is running at https://127.0.0.1:4624
$ CoreDNS is running at https://127.0.0.1:4624/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

To further debug and diagnose cluster problems, use `kubectl cluster-info dump`.

Using `docker ps` you can see the created containers:

 ```sh
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS       PORTS                       NAMES
cd534bf47d15   kindest/node:v1.27.3   "/usr/local/bin/entr…"   3 hours ago   Up 3 hours   127.0.0.1:4624->644/tcp   modern-stack-control-plane
9d7e05f4589a   kindest/node:v1.27.3   "/usr/local/bin/entr…"   3 hours ago   Up 3 hours                               modern-stack-worker
20b2c317ebe2   kindest/node:v1.27.3   "/usr/local/bin/entr…"   3 hours ago   Up 3 hours                               modern-stack-worker2
```

To switching between contexts You can use `kubectl config use-context` with the name of a context that you want to use. To set kubectl context to `kind-modern-stack` use this:

```bsh
kubectl config use-context kind-modern-stack
```

We can see the nodes to created cluster: 

```bsh
kubectl get nodes
```

Some commands to interact with your cluster:

```bsh
# get context
$ kubectx
k3d-onion-dev
kind-modern-stack
minikube
```

## Creating PV and PVC

In Kubernetes, PersistentVolumes (PV) and PersistentVolumeClaims (PVC) manage persistent storage in a cluster. 

- **Persistent Volume (PV)**: A persistent storage unit, such as a hard disk or a cloud storage volume. The cluster administrator provisions a PV and can be used by multiple Pods. A PV has a lifecycle separate from the Pod that uses it.

- **PersistentVolumeClaim (PVC)**: It is a request for persistent storage made by a Pod. A PVC defines the amount of storage and the type of storage (e.g., SSD or HDD) that a Pod needs. When a Pod is created, Kubernetes finds a PV that matches the PVC and mounts the PV into the Pod. Kubernetes will automatically provision a new PV if there is no matching PV.

In summary, a PV is a resource that represents persistent storage in the cluster, and a PVC is a request for persistent storage made by a Pod. Kubernetes automatically manages the allocation and deallocation of PVs based on the PVCs and available PVs in the cluster.

The PV and PVC was provisioned to each cluster with the `pv-config.yaml` and `pvc-config.yaml` from `infra/src/private_volumes` directory.

Create this resources with:

```bash
kubectl apply -f pv-config.yaml
kubectl apply -f pvc-config.yaml
```

Then We can use this resources in Our MinIO pods.

## Installation of DirectPV on Kind Cluster

[DirectPV](https://github.com/minio/directpv?tab=readme-ov-file) functions as a CSI driver specifically crafted for Direct Attached Storage, acting as a distributed persistent volume manager that sets it apart from storage systems like SAN or NAS. Its practicality extends to detecting, formatting, mounting, scheduling, and monitoring drives across multiple servers.

Direct-attached storage (DAS) refers to digital storage directly connected to the computer in use, in contrast to storage accessed via a computer network (such as network-attached storage). DAS comprises one or more storage units like hard drives, solid-state drives, and optical disc drives housed within an external enclosure. The term "DAS" is a retronym, highlighting the distinction from storage area network (SAN) and network-attached storage (NAS).

```sh
# Install DirectPV Krew plugin
kubectl krew install directpv

# Install DirectPV in your kubernetes cluster
kubectl directpv install

# Get information of the installation
kubectl directpv info

# Add drives

## Probe and save drive information to drives.yaml file.
kubectl directpv discover

## Initialize selected drives.
## you can create a yaml file with the configs
cat drives.yaml
kubectl directpv init drives.yaml
kubectl directpv init drives.yaml --dangerous
kubectl directpv list drives
```