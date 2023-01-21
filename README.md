# Projet Kubernetes Learning documentation
## introduction
This page is a documentation of my Kubernetes learning in order to pass the [CKA](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) exam.
#### Outils :
* **Ubuntu_20** 
##  - Installation
You will need to follow the [official documentation](https://kubernetes.io/docs/setup/) to install K8s and his dependancy.

## 1 - Schéma

![](img/architecture_k8s.png)

## Section 2: Core concepts
![](img/core_architecture.png)

### Master 
#### Manage, plan, Schedule and monitor nodes
- ETCD Cluster:
Db that store informations about the cluser in key/value
- Kube Controller Manager: 
Node controller, replication controller etc
- kube-scheduler:
Responsible for scheduling application/container on nodes
- Kube-apiserver:
Responsible for orchestring all operations within the cluster
### Worker Nodes
#### Host Application a Container
- Kubelet:
Lisen for instruction from the kube-apiserver and manager container.
- Kube-proxy:
Enable communication between service within the cluster
- Container runtime (docker, rkt etc):
Run the container

#### ETCD
ETCD records, nodes, pods, configs, secrets, accounts, binding, roles and other.

There is two differents way of deploying ETCD :
 - From scratch, you download the binary and configure it as a service yourself in the master nodes
 - Kubeadm, it deploy it for you as a pod:
 ```bash
 kubectl get pods -n kube-system
 ```
 To list every keys that are store in the ETCD database :
 ```bash
 kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
 ```
 Here are some usefull commands: 
 For example ETCDCTL version 2 supports the following commands:

 ```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
 ```


Whereas the commands are different in version 3

 ```bash
etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put
 ```

To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3


## Section 3: Scheduling

## Section 4: Logging & Monitoring

## Section 5: Application Lifecycle Management

## Section 6: Cluster Maintenance

## Section 7: Security

## Section 8: Storage

## Section 9: Networking

## Section 10: Design and Install a Kubernetes Cluster

## Section 11: Install "Kubernetes the kubeadm way"

## Section 12: End to End Test on  a Kubernetes Cluster

## Section 13: Troublesshooting

## Section 14: Other Topics