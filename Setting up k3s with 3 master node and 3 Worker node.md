# K3S Installation
K3S is a lightweight kubernetes built for IoT and edge computing, provided by the company Rancher. The following picture shows the K3S architecture (source K3S).

<img src="https://picluster.ricsanfre.com/assets/img/how-it-works-k3s-revised.svg">

In K3S all kubernetes processes are consolidated within one single binary. The binary is deployed on servers with two different k3s roles (k3s-server or k3s-agent).

k3s-server: starts all kubernetes control plane processes (API, Scheduler and Controller) and worker proceses (Kubelet and kube-proxy), so master node can be used also as worker node.
k3s-agent: consolidating all kuberentes worker processes (Kubelet and kube-proxy).
Kubernetes cluster will be installed in node1-node5. node1 and node2 will have control-plane role while node3-5 will be workers.

Control-plane node will be configured so no load is deployed in it.


## Master installation (node1-3)
Step 1: Installing K3S control plane node
For installing the master node execute the following command:

```
curl -sfL https://get.k3s.io | K3S_TOKEN=ultraman sh -s - server --write-kubeconfig-mode '0644' --cluster-init --node-taint 'node-role.kubernetes.io/master=true:NoSchedule' --disable 'servicelb' --disable 'traefik' --disable 'local-path' --kube-controller-manager-arg 'bind-address=0.0.0.0' --kube-proxy-arg 'metrics-bind-address=0.0.0.0' --kube-scheduler-arg 'bind-address=0.0.0.0'
```
Where:

* server_token is shared secret within the cluster for allowing connection of worker nodes
* --write-kubeconfig-mode '0644' gives read permissions to kubeconfig file located in /etc/rancher/k3s/k3s.yaml
* --node-taint 'node-role.kubernetes.io/master=true:NoSchedule' makes master node not schedulable to run any pod. Only pods marked with specific tolerance will be scheduled on master node.
* --disable servicelb to disable default service load balancer installed by K3S (Klipper Load Balancer). Metallb will be used instead.
* --disable local-storage to disable local storage persistent volumes provider installed by K3S (local-path-provisioner). Longhorn will be used instead
* --disable traefik to disable default ingress controller installed by K3S (Traefik). Traefik will be installed from helm chart.
* --kube-controller-manager.arg, --kube-scheduler-arg and --kube-proxy-arg to bind those components not only to 127.0.0.1 and enable metrics scraping from a external node.

Installing on node2 and node3
```
curl -sfL https://get.k3s.io | K3S_TOKEN=ultraman sh -s - server --server https://10.1.101.100:6443 --node-taint 'node-role.kubernetes.io/master=true:NoSchedule' --disable 'servicelb' --disable 'traefik' --disable 'local-path' --kube-controller-manager-arg 'bind-address=0.0.0.0' --kube-proxy-arg 'metrics-bind-address=0.0.0.0' --kube-scheduler-arg 'bind-address=0.0.0.0'
```

## Worker Installation

Step 1: Installing K3S worker node

For installing the master node execute the following command:
```
curl -sfL https://get.k3s.io | K3S_URL='https://10.1.101.100:6443' K3S_TOKEN=ultraman sh -s - --node-label 'node_type=worker' --kube-proxy-arg 'metrics-bind-address=0.0.0.0'
```
Where:

* server_token is shared secret within the cluster for allowing connection of worker nodes
* k3s_master_ip is the k3s master node ip
* --node-label 'node_type=worker' add a custom label node_type to the worker node.
* --kube-proxy-arg 'metrics-bind-address=0.0.0.0' to enable kube-proxy metrics scraping from a external node

Step 2: Specify role label for worker nodes

From master node (node1) assign a role label to worker nodes, so when executing kubectl get nodes command ROLE column show worker role for workers nodes.
```
kubectl label nodes <worker_node_name> kubernetes.io/role=worker
```