Kubernetes 

Kubernetes also known as k8s is an open source management system for containerized applications in a clustered environment created by Google.

The unit of execution that Kubernetes works with is the pod. A pod is a collection of containers that share some resources: they have a single IP, and can share volumes. For example, a web server pod could have a container for the server itself, and a container that tails the logs and ships them off to your logging or metrics infrastructure.

At its base, Kubernetes brings together individual physical or virtual machines into a cluster using a shared network to communicate between each server. This cluster is the physical platform where all Kubernetes components, capabilities, and workloads are configured.

The master server in this ecosystem is important since it is the default platform to orchestrate the workload across the nodes.


A node is the other machine in the cluster which is responsible for running workloads. Kubernetes runs apps in containers so each node needs to have a container runtime like Docker configured. The node receives work instructions from the master server and creates or destroys containers accordingly.

Users interact with the cluster by the main API server directly or through clients with libraries. The commands are submitted in JSON or YAML files.

Master Server Components

The master server acts as a primary control node for Kubernetes clusters.

etcd


Kubernetes uses etcd as how an application uses a database. Configuration data, state, and metadata are stored in etcd, but Kubernetes is a distributed system that runs on one master and several worker nodes so etcd is a distributed database. 	

Whenever we run `kubectl get deployments` it gets the data from etcd.

etcd is a key-value database just like a NOSQL database but the similarities stop there.

etcd does not have various data types. It is made to store only kubernetes objects. But redis and other key-value stores have data-type flexibility.

Since all the cluster data is stored in etcd, it is important to always have a backup plan to restore it in case of emergency.

kube-apiserver

This is the main management point of the entire cluster as it allows a user to configure Kubernetes’ workloads and organizational units. A client called kubectl is available as a default method of interacting with the Kubernetes cluster from a local computer.

kube-controller-manager

Controller manager manages different controllers that regulate the state of the cluster, manage workload life cycles, and perform routine tasks. When a change is seen, the controller reads the new information and implements the procedure that fulfills the desired state.


kube-scheduler

Scheduler is the actual component that assigns workloads to the nodes in the cluster. 

Once a pod is created, the scheduler notices that the pod isn’t assigned to a node and it assings the posd to an available node. Scheduler doesn’t run the pod. The scheduler is also responsible of tracking available capacity on each node and makes sure they aren’t scheduled above their available resources.









Node Server Components

A Container Runtime



The first component is a container runtime. A container runtime is software that executes containers and manages container images on a node. This requirement is satisfied by Docker or rkt.

kubelet

The kubelet is the primary “node agent” that runs on each node.  Kubelet works with Pod Specs. A pod spec is a YAML or JSON object that describes a pod. Kubelet makes sure the pod specs are running and healthy.

kube-proxy

Kube-proxy is one of the most important node components that participates in managing Pod-to-Service and External-to-Service networking. The difference between the kube-proxy and a normal reverse proxy is that the kube-proxy proxies requests to Kubernetes Services and their backend Pods, not hosts.


Kubernetes Objects and Workloads


Instead of managing containers directly, users interact with Kubernetes objects.


Pods

A Kubernetes pod is a or  group of containers that are deployed together on the same host.
Pods are useful because containers in the same pod share their lifecycle and storage resources. 


Replication Controllers and Replication Sets

When working with Kubernetes we mostly will be working with more than one pod. A replication controller is an object that creates identical replicas of pods horizontally by increasing or decreasing the number of copies. It also makes sure the number of pods deployed match the number of pods declared in the configuration.

Replication set is the advanced version of the controller which is beginning to replace the controller over time with easier management of the pods to be managed.

Deployments

Deployments are one of the most common workloads to directly create and manage. 

Deployments are intended to replace Replication Controllers.  They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary.

Everyone running applications on Kubernetes cluster uses a deployment. 
It’s what we use to scale, roll out, and roll back versions of our applications.
With a deployment, we tell Kubernetes how many copies of a Pod we want running. The deployment takes care of everything else.

An example of the deploymen.yaml file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django
  labels:
    deployment: django
spec:
  replicas: 1
  selector:
    matchLabels:
      pod: django
  template:
    metadata:
      labels:
        pod: django
    spec:
      containers:
        - name: django
          image: gitumarkk/k8_django_minikube:part_4
          ports:
            - containerPort: 8000
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: user

            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: password

            - name: POSTGRES_HOST
              value: postgres-service

            - name: REDIS_HOST
              value: redis-service
```



Services

A service is used to allow network access to a set of pods.
An example of service.yaml:

```
kind: Service
apiVersion: v1
metadata:
  name: django-service
spec:
  selector:
    pod: django
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 8000
  type: NodePort


```









Jobs

A Job creates a pod which runs a single task to completion.











































https://matthewpalmer.net/kubernetes-app-developer/articles/how-does-kubernetes-use-etcd.html

https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes
