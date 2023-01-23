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

----
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

#### Kube-apiserver
If you do a kubeadm install, you can find the binary in the kubebernetes release page.
As the ETCD, you will need to configure it as a service in the master node. 

Kube-apiserver is responsible for : 
1. Authentificate User
2. Validate request
3. Retrive Data
4. Update ETCD
5. Schduler
6. Kubelet

Is the only one interacting directly with the ETCD data store.
The other component such as the scheduler, kube-controller-manager and kubelet uses the API server to perform update in the cluser in their respective aera.


#### Kube-controller-manager
- *Watch status*
- *Remediate situation*
In kubernetes terms, a controller is a process that continuously monitor the state of various components within the system and work toward bringind the whole system to the desire functionning state.

Here are two exemple of controllers: 
- **Node-controller**:
The node-controller watch status, remediate situation, node monitor period = 5s, node monitor grace period = 40s, POD eviction timeout = 5 min
- **Replication-controller**: 
It is responsible to monitor the status of replica sets and ensuring that the desired number of PODS are available at all time. If a POD dies, it create another one.

But there is a long list of controllers;
Deployement, CronJob, service-account, nodes, namespaces, endpoints, Job, statefull-Set, Replicaset, PV-protection, PV-Binder, replication.

They are all located as a single process  known as *Kube-controller-manager*.

If you set up the *Kube-controller-manager* with the Kube admin tool, you can view it as a POD with :
```bash
kubectl get pods -n kube-system
``` 
in the kube-system namespace on the master node.
In a non-admin set up, you can inspect the options by viewing the service located : 
```bash
cat /etc/systemd/system/kube-controller-manager.service
or 
ps -aux | grep kube-controller-manager
```

#### Kube-Scheduler
Is only responsible for deciding which pod goes on which node.
It doesn't place them on the node, the kubelet(captain) does.
1. Filter nodes
2. Rank nodes

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