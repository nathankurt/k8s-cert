

# Table of Contents
1. [Table of Contents](#table-of-contents)
2. [Core Concepts](#core-concepts)
   1. [Cluster Architecture](#cluster-architecture)
   2. [ETCD](#etcd)
      1. [ETCD For Beginners](#etcd-for-beginners)
      2. [ETCD in Kubernetes](#etcd-in-kubernetes)
   3. [Kube-API Server](#kube-api-server)
   4. [Kube Controller Manager](#kube-controller-manager)
   5. [Kube Scheduler](#kube-scheduler)
   6. [Kubelet](#kubelet)
   7. [Kube Proxy](#kube-proxy)
   8. [PODs](#pods)
      1. [Recap](#recap)
      2. [How to deploy Pods](#how-to-deploy-pods)
      3. [PODs with YAML](#pods-with-yaml)
   9. [ReplicaSets](#replicasets)
      1. [Creating a Replication Controller](#creating-a-replication-controller)
      2. [Creating a ReplicaSet](#creating-a-replicaset)
      3. [Labels and Selectors](#labels-and-selectors)
   10. [Deployments](#deployments)
      1. [What is a Deployment](#what-is-a-deployment)
      2. [Definition](#definition)
   11. [Helpful Pod Commands](#helpful-pod-commands)
   12. [Namespaces](#namespaces)
      1. [Basics](#basics)
      2. [Resource Limits](#resource-limits)
      3. [DNS](#dns)
      4. [Commands](#commands)
   13. [Services](#services)
      1. [Services Use Case](#services-use-case)
      2. [Service Types - Basics](#service-types---basics)
      3. [NodePort](#nodeport)
      4. [Cluster IP](#cluster-ip)
         1. [Creation](#creation)
   14. [Imperative Commands](#imperative-commands)
      1. [POD](#pod)
      2. [Deployment](#deployment)
      3. [Service](#service)
      4. [Misc.](#misc)
3. [Scheduling](#scheduling)
   1. [Manual Scheduling](#manual-scheduling)
      1. [Labels and Selectors](#labels-and-selectors-1)
      2. [Creating Labels](#creating-labels)
      3. [Selectors](#selectors)
      4. [Annotations](#annotations)
   2. [Taints and Tolerations](#taints-and-tolerations)
      1. [Commands](#commands-1)
      2. [Master Nodes](#master-nodes)
   3. [Node Selectors](#node-selectors)
      1. [Limitations](#limitations)
   4. [Node Affinity](#node-affinity)
      1. [Node Affinity Types](#node-affinity-types)
   5. [Node Affinity Vs Taints and Tolerations](#node-affinity-vs-taints-and-tolerations)
   6. [Resource Requirements and Limits](#resource-requirements-and-limits)
      1. [Resource Limits](#resource-limits-1)
   7. [Daemon Sets](#daemon-sets)
      1. [Use Case](#use-case)
      2. [Creation](#creation-1)
4. [Quick Notes](#quick-notes)
   1. [Editing Pods and Deployments](#editing-pods-and-deployments)
      1. [Edit a POD](#edit-a-pod)
      2. [Edit Deployments](#edit-deployments)
5. [End Table of Contents](#end-table-of-contents)


Core Concepts
=============

## Cluster Architecture


* Two kinds of ships
    * Cargo Ships that does the actual work of carrying containers across the sea
    * Control ships that are responsible for monitoring the cargo ships

* Worker Nodes - 
    * Host application as containers

* Master Node - Control Ship
    * Manage, Plan, Schedule, Monitor Nodes
    * Since there is so much info coming all the time, need to maintain info about different "ships", which container is on which "ship" and what time it was loaded, etc.
    * Stored in a highly available key value store known as *Etcd*
    * *Etcd*: A database that stores information in a key-value format
    * *Scheduler*: When ships arrive, you load containers on them using cranes, the cranes identify the containers that need to be placed on ships. 
      * It identifies the right ship based on its size, its capacity, the number of containers already on the ship. and any other conditions such as the destination of the ship
      * *kube-scheduler*

* Different offices in the dock that are assigned to special tasks or departments
    * i.e. Operations team takes care ship handling, traffic control, etc. deal with issues related to damages, the routes take
    * Cargo team takes care of containers. When containers are damaged or destroyed, they make sure new containers are made available
    * Services office that takes care of the I.T and communications between different ships. 

* Controller Manager: In Kubernetes we have controllers available that take care of different areas. 
  * Node-Controller - takes care of nodes
    * onboarding new nodes to cluster, 
    * handling situations where nodes become unavailable or get destroyed.
  * Replication-Controller - ensures desired number of containers are running at all time.  

* kube-apiserver is primary management component of kubernetes. 
  * Responsible for orchestrating all operations within the cluster. 
  * exposes the Kubernetes API which is used by external users to perform management operations on the cluster as well as the various controllers to monitor the various state of the cluster and make the necessary changes as required by the worker nodes to communicate with the server. and by the worker nodes to communicate with the server. 

* Container Runtime Engine
  * Docker is a popular one
  * Not always docker though, could be things like container-d or rocket.

* The captain of the ship. Every ship has one. Responsible for managing all activities on these ships
  * Responsible for liaising with master ships starting with: 
    * letting the master ship know that they are interested in joining the group
    * receiving the appropriate info about the containers to be loaded on the ship
    * and loading the appropriate containers as required
    * Sending reports back to the master about the status of this ship and status of containers on ship
  * captain of the ship is *kubelet* 
* Kubelet is an agent that runs on each node in a cluster
  * Listens for instructions from the kube-api server and deploys or destroys containers on the nodes as required. 
  * Kube-api server periodically fetches status reports from kubelet to monitor the state of the nodes and containers on them

* *Kube-proxy*: Applications on worker nodes need to be able to communicate with each other
  * I.e. you may have a web server running in a container on one node and a db server on the other node. 
  * How would the web server reach the database server on the other node?
  * Kube-proxy service ensures that the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other. 

![kube-architecture](/images/kube-architecture.jpg)
![kube-architecture2](/images/kube-architecture2.jpg)

## ETCD

### ETCD For Beginners
* *ETCD is a distributed, reliable key-value store that is simple, secure, and fast*
* Key value store
  * A dictionary basically
  * Stores info in the form of documents or pages
* Install ETCD
  * Download Binaries
    * `curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz`
  * Extract
    * `tar xzvfetcd-v3.3.11-linux-amd64.tar.gz`
  * Run ETCD Service
    * `./etcd`
    * Starts a service that listens on port 2379 by default and can then attach any clients to the ETCD service to store and retrieve info
    * Default client that comes with etcd is the etcdctl client. use it to store and retrieve key value pairs
      * `./etcdctlset key1 value1`
      * `./etctl get key1`
        * returns `value1`

### ETCD in Kubernetes 
* ETCD Cluster stores info such as:
  * Nodes
  * PODs
  * Configs
  * Secrets
  * Accounts
  * Roles
  * Bindings
  * Others
* All the info you see when you run the `kubectl` command is from the ETCD server. 
* every change you make to your cluster, such as adding additional nodes, deploying pods or replica sets are updated are in the ETCD server.
  * Change isn't considered completed until it's updated in the ETCD server

* ETCD is deployed differently depending on how you set up your cluster
  * Manual Deployment: 
    * Download binaries 
      * `wget-q --https-only \"https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"`
      * `etcd.service`
        * `--advertise-client-urls https://${INTERNAL_IP}:2379\\`
        * ![etcd-service-manual](/images/etcd-service-manual.jpg)
  * Setup - Kubeadm
    * `kubectl get pods -n kube-system`
      * ![etcd-kubectl](/images/etcd-kubectl.jpg)
    * `kubectlexec etcd-master –n kube-systemetcdctlget / --prefix –keys-only`

* ETCD in HA Environment
  * You will have multiple master nodes in your cluster then you will have multiple ETCD instances spread across the master nodes
  * Set the right parameter `--initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380 \`

## Kube-API Server

* When you run a kubectl command, the kubectl is reaching to the kube-apiserver
  * kube-api server authenticates the request and validates it
  * then retrieves the data from the etcd cluster and responds back with the requested info

* don't really need to use kubectl. could instead invoke api directly by sending a post request
* IE: Creating a pod with POST
  * `curl –X POST /api/v1/namespaces/default/pods ...[other]`
    1. Authenticate User
    2. Validate Request
    3. Retrieve Data
    4. Update ETCD 
    5. Scheduler
    6. Kubelet
   * Returns `Pod created!`
     * API Server creates a POD object without assigning it to a node
     * Updates the info in the ETCD server
     * Updates the user that the POD has been created
     * The scheduler continuously monitors the API Server and realizes there is a new pod with no node assigned
       * Scheduler identifies the right node to place the new POD on and communicates that back to kube-api server
     * API server then updates the info in the etcd cluster. 
     * The api server then passes that information to the kubelet in appropriate worker node.
     * Kubelet then creates the pod on the node and instructs the container runtime engine(docker) to deploy the application image. 
     * Kubelet updates the status back to the API server and the API server then updates the data back in the ETCD cluster. 
   * Kube-apiserver is at the center of all the different tasks that needs to be performed to make a change in the cluster.

* Kube-api server is the only component that interacts directly with the etcd datastore. 

* Setting up the hard way, kube-apiserver is available as a binary in the kubernetes release
  * `wgethttps://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver`
  * `kube-apiserver.service`
    * `--etcd-servers=https://127.0.0.1:2379 \\`
  * Some of the important 
* View APi-server -kubeadm
  * `kubectl get pods -n kube-system`
    * `kube-apiserver-master`
* View api-server options - kubeadm 
  * `cat /etc/kubernetes/manifests/kube-apiserver.yaml`
  * See running processes with `ps -aux | grep kub-apiserver`

## Kube Controller Manager 

* Kube controller manager manages various controllers in kubernetes
* Controller is like an office or department within the master ship 
  * An office for the ships would be responsible for monitoring and taking necessary actions about the ships whenever a new ship arrives or when a ship leaves/gets destroyed.
  * Another office could be one that manages containers of the ship. 

* Officers are 
  1. Continuously on the lookout for the status of the ships 
  2. takes necessary actions to remediate the situation

* *Controller* 
  * A process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state
  * For example - Node Controller is responsible for monitoring the status of the Nodes and taking necessary actions to keep the nodes running.

* Node controller checks the status of the nodes every 5 seconds    
  * That way the node controller can monitor health of nodes. If it stops receiving heartbeat from a node, marked as unreachable.
  * Waits 40 seconds to mark it as unreachable. 
  * If a node is marked unreachable, it gets 5 minutes to come back up
    * if it doesn't, removes the pods assigned to that node and provisions them on the healthy ones if pods are part of a replica set. 

* Replication Controller
  * Responsible for monitoring the status of replica sets and ensuring that the desired number of pods are available at all times within the set
  * If a pod dies, it creates another one. 

* Plenty of types of controllers. Whatever concepts we have seen so far in kubernetes such as deployments, services, namespaces, Persistent volumes, implemented through these controllers

* How do you see these controllers? 
  * In the Kube-Controller-Manager
  * Download the kube-controller-manager and run it as a service
    * `wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager`
    * `kube-controller-manager.service`
    * Option called controllers to see which one is enabled. by default, they are all enabled. 

  * Or with kubeadm
    * `kubectl get pods -n kube-system`
      * Deploys kube-controller-manager as a pod in the kube-system namespace on the master node. 
      * can view options with `cat /etc/kubernetes/manifests/kube-controller-manager.yaml`
  * Without you can run `cat /etc/systemd/system/kube-controller-manager.service` 

* See running processes with `ps -aux | grep kube-controller-manager`


## Kube Scheduler

* Scheduler is only responsible for deciding which pod goes on which node. It doesn't actually place the pod on the node
  * That's the kubelet.

* Why do you need a scheduler? 
  * want to make sure the right container ends up on the right ship. 
  * Want to make sure containers are placed on right ship so they go to the right place. 

* Scheduler looks at each pod and tries to find the best node for it. 
  * Two Phases:
    * First Phase - Filter Nodes:
      * Filter out nodes that don't fit profile of pods i.e. Nodes that don't have sufficient CPU and memory resources requested by the pod.
    * Second Phase - Rank Nodes: 
      * Uses a priority function to assign a score to the nodes on a scale of 0 to 10. That would be free on the nodes after placing the pod on them 
        * So whichever has more resources would be placed on. 

* Installing Kube-scheduler 
  * download binary and run it as service. 
    * `wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler`
    * `kube-scheduler.service`


## Kubelet
* Kubelet is like the captain on the ship
* Lead all activities on the ship!
* Send reports at regular intervals on the status of the ship and the containers on them.


* Kublet in the kubernetes worker node, registers the node with the kubernetes cluster. 
* When it receives instructions to load a container or a POD on the node, it requests the container run time engine(docker) to pull required image and run an instance. 
* Kubelet then continues to monitor the state of the POD and the containers in it and reports to the kube-api server on a timely basis

1. Register Node
2. Create PODs
3. Monitor Node & PODs

* Install kubelet:
  * Kubeadm does not deploy kubelets.
  * You must always manually install kubelet on the worker nodes.
    * `wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubele`
    * `kubelet.service`
  * View Kublet options
    * `ps -aux | grep kubelet`

## Kube Proxy

* Every pod can reach every other pod.
* This is accomplished by deploying a POD networking solution to the cluster.
  * POD Network - an internal virtual network that spans across all the nodes in the cluster through which all the pods connect to. 
  * Many solution available to deploying such a network
* Example: In this case, I have a web application deployed on the first node and a database application deployed on the second.
  * Web app can reach the database by using the IP of the db pod. 
  * No guarantee that the ip of the database pod will always remain the same.
  * Better way for web app to access the db is using a service. so we create a service to expose the db application across the cluster. 
  * The service cannot join the pod network because the service is not an actual thing. 
    * Not a container like pod so it doesn't have any interfaces or an actively listening process. virtual component that only lives in the kubernetes memory. 
  * Kube-proxy is a process that runs on each node in the kubernetes cluster
    * Job is to look for new services and every time a new service is created, it creates the appropriate rules on each node to forward traffic to those services to the backend pods
    * One way it does this is by using IPTABLES rules
      * In this case it creates an IP tables rule on each node in the cluster to forward traffic heading to the IP of the service which is 10.96.0.12to the IP of the actual pod which is 10.32.0.15
  * ![pod-network](/images/pod-network.jpg)

* Installing Kube-proxy
  * `wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy`
  * or `kubectl get pods -n kubesystem` with kubeadm.
  * `kubectl get daemonset -n kube-system`

## PODs

### Recap

* With kubernetes, ultimate aim is to deploy our application in the form of containers on a set of machines that are configured as worker nodes in a cluster. 
* Kubernetes does not deploy containers directly on worker nodes.
  * Instead the containers are encapsulated into a kubernetes object known as PODs.
  * **Pod:** Single instance of an application
    * Smallest thing you can create in kubernetes
* Simplest of simple instance one  instance of application running in a single docker container encapsulated in a POD
  * As it scales, spin up additional instances by creating a new pod with a new instance of the same application
  * If user base further increases and current node has no sufficient capacity, create a new pod on a new node in the cluster. 
  * PODS usually have a 1 - 1 relationship
  * To scale up, you create new pods
    * Scale down delete pods

* Multi-Container PODs
  * A single pod can have multiple containers except for the fact that they're usually not multiple containers of the same kind
  * If intention was to scale our application, then we would need to create additional pods
  * **Helper Container**
    * supporting task for our web app such as processing a user entered data processing a file uploaded by the user etc. and want helper containers to live alongside application container
      * In that case you can have both of these containers part of the same pod so that when a new application container is created, the helper is also created and when it dies, the helper also dies since they are part of the same pod. 
      * The two containers can also communicate with each other directly by referring to each other as localhost. 
      * Can also share same storage space

### How to deploy Pods
* `Kubectl run nginx` 
  * deploys a docker container by creating a POD. So first creates a POD automatically and deploys an instance of the nginx docker image. 
    * Where does it get app image from? 
      * `--image nginx` command so `kubectl run nginx --image nginx`
      * See pods available with `kubectl get pods`


### PODs with YAML

  * YAML in kubernetes
  * pod-definition.yml
  * ```yaml
      apiVersion:
      kind: 
      metadata:

      spec:
    ```
*  all required fields!!
    * `apiVersion: v1` could also be `apps/v1`
    * kind could be `POD`, `Service`, `ReplicaSet`, `Deployment`
    * **metadata**:
    * ```yaml
        metadata: 
          name: myapp-pod
          labels: 
              app: myapp
              type: front-end   
        ```
      
  * Metadata is a dictionary. number of spaces doesn't matter but they should stay the same since they are siblings
     * cannot add any other property that you want in metadata. 
     * For spec, refer to documentation since there are plenty. With app that has a single container though, not too tough
     * ```yaml
        spec:
          containers:
          - name: nginx-container
            image: nginx
        ```
    * dash(-) indicates that it is a list and first item in the list. 
    * when yaml is made, run `kubectl create -f pod-definition.yml`
    * Delete pod with `kubectl delete myapp-pod`
    * Once pod is created, run `kubectl get pods` to view pods available.
    * to see detailed info about pod run `kubectl describe pod myapp-pod` 
 
## ReplicaSets

   * They are the processes that monitor kubernetes objects and respond accordingly. 
  
   * Why do we need replica set
     * If something happens and pod fails, users will no longer be ale to access our application
     * to prevent users from losing access to our app, we would like to have more than one instance or pod running at the same time. 
     * Replication controller helps us run multiple instances of a single pod in the kubernetes cluster and providing high availability
   * Can you use replication controller if you plan on using a single pod? 
     * **NO** Even if you have a single pod, the replication controller can help by automatically bringing up a new pod when the existing one fails. 
     * Thus the replication controller ensures that the specified number of parts are running at all times. 

   * Also need to create multiple pods to share load across them. 
   * ![replication-controller](/images/replication-controller.jpg)

   * Two similar terms for when the demand increases both have same purpose but not the same. 
     * **Replication Controller** 
       * The older tech that's being replaced by replica set. 
     * **Replica Set** 
       * New recommended way to set up replication

### Creating a Replication Controller

   * `rc-definition.yml`
   * ```yaml
      apiVersion: v1
      kind: ReplicationController
      metadata:
        name: myapp-rc
        labels:
          app: myapp
          type: front-end

      spec:
        template:
          metadata:
            name: myapp-pod
            labels:
              app: myapp
              type: front-end
          spec:
            containers:
              - name: nginx-container
                image: nginx
        
        replicas: 3


        ```
  * create template section under spec to provide a part template to be used by the replication controller
  * move all the contents of the pod-definition file except for the first few lines and put it in the rc-definition file
  * Replication controller is parent, pod definition is child
  * for replica count, add replicas tag to spec and input the number of replicas you'll need under it. 
  * then run `kubectl create -f rc-definition.yml`
    * when created, it also creates the pods.
  *  to see replicas, run `kubectl get replicationcontroller` command and it will give you number of replicas. 

### Creating a ReplicaSet

`replicaset-definition.yml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
  

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  
  replicas: 3
  selector:
    matchLabels:
      type: front-end

```

* Need to have the apps/ or you will get an error: unable to recognize replicaset
* ReplicaSet needs `selector:` 
* Can also manage parts that were not created as part of the replica at creation
  * i.e. The reports created before the creation of the replica set that match labels specified in the selector.
  * Replica set will also take those pods into consideration when creating the replicas. 
* can still use `selector:` in replicacontroller but just assumes it is the same as the labels provided in the part definition file
* Required for ReplicaSet and has to be written in form of matchLabels: 
  * Simply matches the labels specified under it to the labels on the pod. 
  * Replica set selector also provides many other options for matching labels that were not available in the replication controller

* to create run `kubectl create -f replicaset-definition.yml`
  * to see list of pods, run `kubectl get replicaset`

### Labels and Selectors
* How ReplicaSet knows which pods to monitor
* Labeling comes in handy here. 
  * Can provide labels as a filter for replica set
  * under the selector section we use `matchLabels` and provide the same label we used while creating the pod. 

* Scale: 
  * Update the number of replicas in the definition files to 6
    * Run `kubectl replace -f replicaset-definition.yml` to update the replicaset to have 6 replicas
  * run `kubectl scale --replicas=6 -f replicaset-definition.yml` or `kubectl scale --replicas=6 myapp-rep`
    * using file name as input will not result in the number of replicas being updated automatically

* Commands
  * `kubectl create -f replicaset-definition.yml` creates replicaset
  * `kubectl get replicaset` see list of replicasets created
  * `kubectl delete replicaset myapp-replicaset` *also deletes all underlying pods.
  * `kubectl replace -f replicaset-definition.yml` to update it after making changes
  * `kubectl scale --replicas=6 -f replicaset-definition.yml` 
  * `kubectl edit replicaset new-replica-set` modifies the image, you can save out of tmp by running `:w [filename]`

## Deployments

### What is a Deployment

* Things you want in a deployment: 
  * Say you have a web server that needs to be deployed in a production environment
  * You need not one, but many instances running
  * Whenever newer versions or builds become available on the docker registry, you would like to upgrade your builds seamlessly
    * But not all at the same time since it may impact users
    * may want to upgrade one after the other
  * Would like to be able to roll back
  * Would like to make multiple changes to environment, don't want to make changes immediately
    * but apply a pause to your environment, make changes and then resume so that all changes are rolled out at the same time.
* **Kubernetes can do that** 

* Pods -> Replica Set -> Deployment
  * Deployment can update underlying instances seamlessly. 
  * ![deployment](/images/deployment.jpg)

### Definition
Same as replica set except kind will now be `Deployment`
`deployment-definition.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
          - name: nginx-container
            image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```
* `kubectl create -f deployment-definition.yml`
* `kubectl get deployments`
* `kubectl get replicaset` will get the replicaset in the deployment
* `kubectl get pods` 
* Basically the same as replicaset except that it created a deployment object.
* `kubectl get all` shows all created objects at once. 

## Helpful Pod Commands
* Create an NGINX Pod
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx`
* Generate POD Manifest YAML file (`-o yaml`) Don't create it(`--dry-run`)
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`
* Create a deployment
  * `kubectl run --generator=deployment/apps.v1 nginx --image=nginx`
* Generate Deployment YAML file (`-o yaml`). Don't create it(`--dry-run`)
  * `kubectl run --generator=deployment/apps.v1 nginx --image=nginx --dry-run -o yaml`
* Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
  * `kubectl run --generator=deployment/apps.v1 nginx --image=nginx --dry-run --replicas=4 -o yaml`
* Save it to a file - (if you need to modify or add some other details before actually creating it) 
  * `kubectl run --generator=deployment/apps.v1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml`


## Namespaces


### Basics

* Two boys named Mark, to address each other, they use last names. 
  * In their house though with families, they address by first name because there isn't another mark there. 
  * ![namespace-houses](/images/namespaces-houses.jpg)

* So far we've been doing everything in the default namespace. 
  * Created automatically when the cluster is first set up
* When cluster is first set up, kubernetes creates a set of pods and services for its internal purpose such as those required by the networking solution, dns service. etc. to isolate them from the user to prevent you from accidentally deleting or modifying the services. 
  * Created under namespace *kube-system* 
* *kube-public*
  * where resources that should be made available to all users are created. 

* If your environment is small you shouldn't really have to worry about namespaces. 
  * As you grow though, the use of namespaces is very important

* Create namespaces too 
  * IE namespace for dev and prod so you don't accidentally modify a resource in production when working in dev. 

### Resource Limits

  * You can also assign a quota of resources to each of these namespaces so it's guaranteed a certain amount and doesn't use more than it's allowed.
  * ![resource-limits](/images/resource-limits.jpg)

### DNS

* can communicate with the same server easily. But if you want to communicate with something outside of the namespace, you must append the name of the namespace to the name of the service
  * Use the `servicename.namespace.svc.cluster.local` format. In this case it would be `db-service.dev.svc.cluster.local`

* You're able to do this because when the service is created, a DNS entry is added automatically in this format.
* `cluster.local` is the default domain name of the kubernetes cluster. `SVC` is the subdomain. 
  * ![DNS-format](/images/DNS-format.jpg)

### Commands
  * Get pods in the kube-system namespace
    * `kubectl get pods --namespace=kube-system`
  * Create pod in dev namespace
    *  `kubectl create -f pod-definition.yml --namespace=dev`
    *  Can also move namespace into the pod definition file under metadata section like: 
      ```yaml
      apiVersion: v1
      kind: Pod

      metadata:
        name: myapp-pod
         ### LIKE THIS #######
        namespace: dev
         ##################### 

        labels:
          app: myapp
          type: front-end
      spec:
        containers:
          - name: nginx-container
            image: nginx
      ```
  * Create Namespace Like
   `namespace-dev.yml`
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ```
    * `kubectl create -f namespace-dev.yml`
    * `kubectl create namespace dev`
  * Switch default namespace to dev
    * `kubectl config set-context $(kubectl config current-context) --namespace=dev`
  * get pods from all namespaces
    * `kubectl get pods --all-namespaces`
  

  * Create a Resource Quota
    * ```yaml
      apiVersion: v1
      kind: ResurceQuota
      metadata:
        name: compute-quota
        namespace: dev
      
      spec:
        hard:
          pods: "10"
          requests.cpu: "4"
          requests.memory: 5Gi
          limits.cpu: "10"
          limits.memory: 10Gi
      ```
    * `kubectl create -f compute-quota.yaml`

## Services

* enable communication between various components within and outside of the application. 
* Help connect apps together with other apps or users

* IE. App has groups of pods running various sections such as 
  * a group for serving front end load to users 
  * Group for running back end processes
  * Third group connecting to an external data source

* Enable loose coupling between microservices in our application

### Services Use Case

  * External Communication
    * Deployed Pod having web app running on it
    * How do we as an external user access the web page?
      * Kubernetes Node has an IP address that is `192.168.1.2`
      * Laptop is on the same network as well so has IP address `192.168.1.10`
      * The internal POD network is in the range `10.244.0.0` and has an `IP 10.24.0.2`
        * Clearly cannot ping or access the POD at address `10.244.0.2` since it's in a separate network. 
      
      * Things we could do: 
        * SSH into the kubernetes node at `192.168.1.2` and then from the node, we would be able to access the POD's webpage by doing a curl. Or if the node has a GUI, we could fire up a browser and see the webpage from `http://10.244.0.2`
          * ![external-service](/images/external-service.jpg)
        * That's not what we want though, we want to be able to access the webpage simply by accessing the IP of the kubernetes node. 
      
      * This is where kubernetes service comes into play. 
        * Kubernetes service is an object just like PODs, ReplicaSet or Deployments that we worked with earlier. 
        * Can listen to a port on the Node and forward requests on that port to a port on the POD running the WebApp.

### Service Types - Basics

1. NodePort
   * Makes an internal POD accessible on a Port on the Node.
2. ClusterIP
   * The service creates a virtual IP inside the cluster to enable communication between different services such as a set of front end servers to a set of back end servers. 
3. LoadBalancer
   * Provisions a load balancer for our service in supported cloud providers.
   * Distribute load across the different web servers in your front end tier. 

  ![services-types](/images/services-types.jpg)


### NodePort

   * Three ports involved:
    * Port on the POD where the actual web server is running is **80**
      * Also referred to as the **targetPort**
    * Port on the service itself is just called the **Port**
      * Terms are from the viewpoint of the service.
    * Port on the node itself is running on 300008
      * **NodePort** 
      * Port range from `30000 - 32767`
   * ![NodePort](/images/Service-NodePort.jpg)

   * How to create service
    * `service-definition.yml`
    * ```yaml
       apiVersion: v1
       kind: Service
       metadata:
         name: myapp-service
         ### Can Have Label but don't need that
       
       spec:
         type: NodePort
         ports:
           - targetPort: 80 #an array
             port: 80 #Only mandatory field
             nodePort: 30008
         selector:
           #provide a list of labels and pull values
           #from that in pod metadata section
           app: myapp
           type: front-end
      ```

    * When done run `kubectl create -f service-definition.yml` to create
    * `kubectl get services` to get the services that are running. 
      * Will give you a `CLUSTER-IP` IP and you can now access that IP using curl or web browser
        * `curl http://192.168.1.2:30008`

   * What do you do when you have multiple pods? 
     * We have multiple similar pods running our web application. they all have the same labels with a key app and set to value `myapp`
       * same label is used as a selector during the creation of the service. 
     * So when service is created, it looks for a matching pod with the label and finds three of them
     * Service then automatically selects all the three pods as endpoints to forward the external requests. 
     * **NO ADDITIONAL CONFIG REQUIRED** 

   * What about when the web application is on pods in separate nodes in the cluster
     * When we create a service, without any additional config, Kubernetes creates a service that spans across all the nodes in the cluster and maps the target port to the same node port on all the nodes in the cluster. 
     * This way you can access your application using the IP of any node in the cluster and using the same port number which in this case is 30008
     * ![multiple-clusters-node-port](/images/multiple-services-nodeport.jpg)

   * No matter what, service is created exactly the same. Making it highly flexible and adaptive. 

### Cluster IP

* May have a number of pods: 
  * You may have a number of pods running a front end web server
  * another set of pods running a back end web server. 
  * Another set of PODs running a key-value store like Redis
  * and another set of PODs running a persistent database like MySQL

* Web front end server needs to communicate to the back end servers and redis server, etc.

* Best way to establish connectivity between these services or tiers of application?
  * Also know that the pods all have an IP address assigned to them as we can see on the screen. 
  * But these IPs aren't static. so can't rely on those IP addresses for internal communication between the application. 
  * A kubernetes service can help us group these PODs together and provide a single interface to access the pods in a group. 

  * For example, a service created for the backend PODs will help group all the backend pods together and provide a single interface for other pods to access the service. 
  * Requests are forwarded to one of the PODs under the service randomly. 
  * Similarly create additional services for Redis and allow the backend parts to access the redis system
  * Each layer can now scale or move as required without impacting communication between the various services. 
  * Each service gets an IP name assigned to it inside the cluster and that is the name that should be used by other pods to access the service. 
    * ![cluster-ip](/images/Cluster-IP.jpg) 

* Type of service is known as **cluster IP**

#### Creation
* `service-definition.yml`
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: back-end

  spec:
    type: ClusterIP ### The default type anyways
    ports:
      - targetPort: 80
        port: 80 
    
    selector:
      app: myapp
      type: back-end

  ```
* can create service using `kubectl create -f service-definition.yml`
* The service can be accessed by other PODs using the ClusterIP or the service name. 
  * `kubectl get services`  



## Imperative Commands

* `dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run option. This will not create the resource, instead, tell you weather the resource can be created and if your command is right.
  
* `-o yaml`: This will output the resource definition in YAML format on screen.

### POD

* **Create an NGINX Pod**
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx`
* **Generate POD Manifest YAML file (`-o yaml`) Don't create it (`--dry-run`)**
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`
* **Generate NGINX POD with labels set to tier=db**
  * `kubectl run --generator=run-pod/v1 nginx --image=nginx -l tier=db`

### Deployment

* **Create a deployment**
  * `kubectl create deployment --image=nginx nginx`
* **Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**
  * `kubectl create deployment --image=nginx nginx --dry-run -o yaml`
* **Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)**
  * `kubectl run --generator=deployment/apps.v1 nginx --image=nginx --dry-run --replicas=4 -o yaml`
    * The usage --generator=deployment/v1beta1 is deprecated as of Kubernetes 1.16. The recommended way is to use the kubectl create option instead.
  * **NOTE:** kubectl create deployment does not have a `--replicas` option. You could first create it and then scale it using the `kubectl scale` command. 
    * `kubectl create deployment --image=nginx nginx`
    * `kubectl scale deployment.apps/nginx --replicas=x` 
      * This one is a maybe.. will try it out
*  **Save it to a file - (If you need to modify or add some other details)**
   *  `kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml`
      *  You can then update the YAML file with the replicas or any other field before creating the deployment. 

### Service

  * **Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**
    * `kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml`
      * This will auto use the pod's labels as selectors.
    
    OR

    * `kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml` 
      * (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
  
  * **Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**
    * `kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml`
      *  (This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

    OR 

    * `kubectl create service nodeport nginx -tcp=80:80 --node-port=30080 --dry-run -o yaml`
      * (This will not use the pods labels as selectors)
    * Both the above commands have their own challenges. While create service cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.


### Misc.

  * Create a service of type ClusterIP with the port 6379, 

  * **Create a deployment named webapp using the image `kodekloud/webapp-color` with 3 replicas**
    * Create Deployment with `kubectl create deployment webapp --image=kodekloud/webapp-color`
      * Scale with `kubectl scale deployment.v1.apps/webapp --replicas=3`
  
  
  * **Expose the webapp as service webapp-service application on port 30082 on the nodes on the cluster** 
    * First run `kubectl expose deployment webapp --type=NodePort --port=8080 --name=webapp-service --dry-run -o yaml > webapp-service.yaml` To create the deployment and set the selector to the deployment "webapp" (This adds the endpoints you need)
      * Then edit the file and add the line `nodePort: 30082` in the ports spec. 
    
    
    * You could also do `kubectl create service nodeport webapp-service --tcp=8080 --node-port=30082 --dry-run -o yaml > create-service.yaml` 
      * Then go and change the `selector: app: webapp-service` to `selector: app: webapp`










# Scheduling


## Manual Scheduling

* What do you do when you don't have a scheduler in your cluster? 
  * don't want to rely on the built in scheduler and want to schedule the pods yourself. 

* How does it work? 
    * start with `pod-definition.yaml`
    * ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: nginx
        labels:
          name: nginx
      spec:
        containers:
          - name: nginx
            image: nginx
            ports:
              - containerPort: 8080
         ### Not set by default
        nodeName:
         ###########
      ```
    * Every pod has a field called `NodeName` that, by default is not set.
    * don't typically specify it since kubernetes does it automatically. 
    * Scheduler goes through all the pods and looks for those that don't have this property set. 
      * Those are the candidates for scheduling
    * Then identifies the right node for the POD by running the scheduling algorithm.
    * Once identified, it schedules the pod on the node by setting the `nodeName` property to the name of the node by creating a binding object.
    * ![how-scheduling-works](/images/how-scheduling-works.jpg)

* If **No Scheduler**
  * Pods continue to be in pending state
  * So you can manually assign pods to nodes yourself
    * Set `nodeName` field to name of node yourself 
       ![manual-schedule](/images/manual-schedule.jpg)
    * Can only specify node name at creation time. 

  * **What if Pod is already created and you want to assign the pod to a node?**
    * kubernetes doesn't allow you to edit the nodeName property of a pod
    * So create a `binding object` and send a post request to the pod binding API, thus mimicking what the actual scheduler does. 
    * `pod-bind-definition.yaml`
    * ```yaml
      apiVersion: v1
      kind: Binding
      metadata:
        name: nginx
      target: #NEW STUFF
        apiVersion: v1
        kind: Node
        name: node02
      ```
    * Then send a post request to the pods binding API with the data set to the binding object in a JSON format. 
      * Must convert the YAML file into its equivalent JSON format. 
        * `curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind": "Binding“ .... }'`
          * returns `http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/`

### Labels and Selectors


  * Best way to group things together and filter them based on your need is with Labels
  * Labels are properties that you can add to each item for their class, kind, and color etc.
    * Selectors help you filter these items.

  * For each object attach labels as per your needs
    * app, function, etc.
  * Then while selecting, specify a condition to filter specific objects.
    * `app = App1`

### Creating Labels

Specify Labels: you can add as many as you like
  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  ### Right Here
  labels:
    app: App1
    function: Front-end
  ####################
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080

```

### Selectors

* Select Pod(once the pod is created):
  * `kubectl get pods --selector app=App1`
    * If you want to select multiple pods, separate them with a comma but no spaces
    * `kubectl get pods --selector env=prod,bu=finance,tier=frontend`

* Kubernetes objects use labels and selectors internally to connect different objects together. 
  * For example, to create a replicaset consisting of 3 different consisting of three different pods. 
    * First label the pod definition and use selector in a replicaset to group the pods. 
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      ### Labels here are labels of replicaset itself
      labels:
        app: App1
        function: Front-end
    spec:
      replicas: 3
      selector:
        # match labels here
        matchLabels:
          app: App1
      template:
        metadata:
          #### Labels defined here are
          #### labels configured on the pods
          labels:
            app: App1
            function: Front-end
        spec:
          containers:
            name: simple-webapp
            image: simple-webapp
    ```
    * Not too concerned with labels of replicaset right now because we are trying to get the replica set to discover the pods
      * labels on replicaset will be used if you were to configure some other object to discover the replica set. 
    
    * In order to connect the replica set to the pod,we configure the `selector` field under the replicaset spec to match the labels defined on the pod.  
      * a single label will do if it matches correctly
      * However, if you feel there could be other pods with same label but different function, then you could specify both the labels to ensure that the right pods are discovered by the replica set
    * On creation, if the labels match, the replica set is created successfully. 

### Annotations

* Used to record other details for informatory purposes  
  * i.e. tool details: name, version, build information etc.
  * contact details, phone numbers, email ids etc. that may be used for some kind of integration purpose. 

`replicaset-definition.yaml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  #New thing here
  annotations:
    buildversion: 1.3
  ################
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        name: simple-webapp
        image: simple-webapp
```

## Taints and Tolerations

* Bug approaching a person analogy
  * To prevent bugs from landing on a person, we spray the person with bug spray(**Taint**).
  * The bug is intolerant to the smell. So when approaching the person, the taint applied on the person throws the bug off.
    * However there could be other bugs that are tolerant to the smell and so the taint doesn't really affect them and end up landing on the person. 
  * Two things that decide if a bug can land on a person:
    1. The Taint of the person
    2. The bug's toleration level to that particular taint

Back to Kubernetes...
* The person is a node and the taints are pods
* Taints and tolerations are used to set restrictions on what parts can be scheduled. 

**Example**
* Start with a cluster with three worker nodes named `one` `two` and `three`
* Also have a set of pods that are to be deployed on these nodes, `A`,`B`,`C`, and `D`
* When the pods are created, kubernetes scheduler tries to place these parts on the available worker nodes.
  * When no restrictions, scheduler balances the pods across all of the nodes to balance them out equally. 
  * ![scheduler-no-restrictions](/images/scheduler-no-restrictions.jpg)


* Now lets assume that we have dedicated resources on `Node 1` for a particular use case or application.
  * So we would like only pods that belong to application to be placed on `Node 1`
* First we prevent all pods from being placed on the Node by placing a taint on the Node.
  * call it `blue`
* By default pods have no tolerations
  * Unless specified otherwise, none of the parts can tolerate any taint. 
    * Right now, no pods can be placed on `Node 1` because none of them can tolerate the taint, `blue`
* Next we must specify which pods are tolerant to particular taint.
  * We want only pod `D` to be able to be placed in `Node 1`
  * We give Pod `D` a toleration to blue
* Pod `D` is now tolerant to blue so when the scheduler tries to place this part on `Node 1`, it goes through. 
* Here's how it would work now.
  1. Scheduler tries to place `Pod A` on `Node 1`, but because taint, is thrown off and placed on `node 2`
  2. Scheduler tries to place `Pod B` on `Node 1`, thrown off again because taint. placed on next free node, which is `Node 3`
  3. Scheduler tries to place `Pod C` on `Node 1`, thrown off again and placed on `node 2`
  4. Scheduler tires to place `Pod D` on `Node 1`, since it's tolerant it accepts and is placed. 
  ![taint-tolerant](/images/taint-tolerant.jpg)

* **Remember:**
  * _Taints_ are set on _Nodes_
  * _Tolerants_ are set on _Pods_

### Commands

* Add Taint - Nodes
  * `kubectl taint nodes node-name key=value:taint-effect`
    * specify name of node to taint 
    * followed by taint itself, which is a key value pair. 
    * Then taint-effect(what happens to pods that do not tolerate this taint)
      * Three effects: 
        1. `NoSchedule`:
           * Pods will not be scheduled on Node
        2. `PreferNoScheduler`:
           * System will try to avoid placing pod on node, but not guaranteed
        3. `NoExecute`:
           * New pods will not be scheduled on node and existing pods will be evicted
    * `kubectl taint nodes node1 app=blue:NoSchedule`
  
* Add Toleration - PODS 
  * `pod-definition.yaml`
    * ```yaml
        apiVersion:
        kind:
        metadata:
          name: myapp-pod
        spec:
          containers:
            - name: nginx-container
              image: nginx
           #Values need to be enclosed with double quotes
          tolerations:
            - key: "app" #The key in key=value
              operator: "Equal" #The = in key=value
              value: "blue" #the value in key=value
              effect: "NoSchedule" # the taint effect
           #########################
      ```
  * `NoExecute` taint effect:
    * When pod is evicted, it's killed. 

* **Taints only Stop pods from coming in, it doesn't force pods to go to certain nodes**
  * Just because `Node 1` has a `blue` taint and `Pod D` has a tolerance for `blue`, doesn't mean that `Pod D` will always be placed in `Node 1`
     ![taint-not-placed](/images/taints-not-placed.jpg)
    * That is called [Node Affinity](#node-affinity)


### Master Nodes
* So far we've only been referring to worker nodes. but we also have master nodes in the cluster
  * technically just another node that has all the capabilities of hosting a pod plus runs all the management software.
  * Scheduler doesn't schedule any pod on the master node.
    * Taint is set on the master node automatically that prevents any parts from being scheduled on this node.
    * ![master-taint](/images/master-taint.jpg)
  * Best practice is to not deploy application workloads on a master server. 
    * To see taint: 
      * `kubectl describe node kubemaster | grep Taint`
    * To remove taint from master which has the effect of NoSchedule:
      * `kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-`


## Node Selectors

* **Example**: 
  * You have three node cluster of which two are smaller nodes with lower hardware resources
    * One is a larger node configured with more resources.
  * Different workloads running in your cluster
    * Would like to dedicate the data processing workloads that require higher horsepower to the larger as that is the only node that will not run out of resources in case the job demands extra. 

  * In the current default setup, any pods can go to any nodes. 
    * So data processing pod, could very well end up on lower-end nodes which is not desired. 

  * To solve, we can set a limitation on the pods so that they only run on particular nodes. 
    * With **Node Selectors**
      * `pod-definition.yml`
        * ```yaml
          apiVersion:
          kind:
          metadata:
            name: myapp-pod
          spec:
            containers:
              - name: data-processor
                image: data-processor
             # New Property to add
            nodeSelector:
              #key value pair of size:large are labels the pod use. 
              #to use labels in nodeSelector, must have first labeled nodes
              #prior to creating pod
              size: Large # need to go and make that. 
          ```
            * `kubectl create -f pod-definition.yml`

      * **Label Nodes**
        * `kubectl label nodes <node-name> <label-key>=<label-value>`
        * i.e. `kubectl label nodes node-1 size=Large`

### Limitations
  * If you want something more complex like
    * Place the pods on Large or Medium
    * Place the pods on any nodes that are not small.
  * Can't do it at all. 

## Node Affinity
  * Primary purpose is to ensure pods are hosted on particular nodes. 
    * like ensure large processing pod ends up on `large node1`
    * Can provide advanced capabilities to limit pod placement on specific nodes.
    * Much more complex though.
  * `pod-definition.yml`
    * ```yaml
      apiVersion:
      kind:
        ## SAME AS PREVIOUS nodeSelector One
      metadata:
        name: myapp-pod
      spec:

        containers:
          - name: data-processor
            image: data-processor
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
                - matchExpressions:
                  - key: size
                    #For doing the advanced stuff, 
                    operator: In #NotIn will match the node with a size not set to large. 
                    # operator: Exists will just check if label exists and don't need values.
                    # There are more but check docs. 
                    values:
                      - Large
                      # - Medium to add value
      ```
      
  * What if someone changes the label on the node at a future point in time? Will pod continue to stay on Node? 
    * Answered by long `requiredDuringSchedulingIgnoredDuringExecution` which is affinity types

### Node Affinity Types
  * Defines the behavior of the scheduler with respect to node affinity and the stages in the lifecycle of the pod. 
  * Currently two types but plan to offer more. 
    * `requiredDuringSchedulingIgnoredDuringExecution`
    * `preferredDuringSchedulingIgnoredDuringExecution`
    * **Planned Release**: `requiredDuringSchedulingRequiredDuringExecution`


    * There are two states in the lifecycle of a pod when considering node affinity. 
      * `During scheduling`
        * The state where a pod does not exist and is created for the first time
        * No doubt that when a pod is first created, the affinity rules specified are considered to place the pod on the right node.
        
        
         * If you select the required type(`requiredDuringSchedulingIgnoredDuringExecution`), the scheduler will mandate that the pod be placed on a node with the given affinity rules
          * If it can't find one, the pod will not be scheduled.
          * Used where placement of pods is crucial.  
        * If you select the preferred type(`preferredDuringSchedulingIgnoredDuringExecution`), the scheduler will simply ignore node affinity rules and place the pod on any available node
          * Way of telling the scheduler, "try you're best to place the pod on the matching node, but if you can't find one, just place it anywhere."  
  
      * `During Execution`
        * the state where a pod has been running and a change is made in the environment that affects node affinity such as a change in the label of a node.
          * For example, say admin removed the label we said earlier called `size=large` from the node. 
          * What happens to pods running on the node?
            * Currently, both affinity's are set to ignored
              * Pods will continue to run and any changes in node affinity will not impact them once they are scheduled. 
            * Planned new types will allow for `required` during execution
              * pod running on the large node will be evicted or terminated if the label `large` is removed.
 

**Node Affinity Types Chart**

  |         | DuringScheduling | During Execution |
  | ------: | ---------------: | ---------------: |
  |  Type 1 |         Required |          Ignored |
  |  Type 2 |        Preferred |          Ignored |
  | Type 3* |         Required |         Required |

  * *Type 3 is planned but not a thing yet*
 

  **Get Pods and which node they are assigned to command**
    * `kubectl get pod -o=custom-columns=NODE:.spec.nodeName,NAME:.metadata.name | sort`
   

## Node Affinity Vs Taints and Tolerations

* Start with an exercise
  * Three nodes and three pods each in three colors, `blue`,`red`, and `green`
  * Goal is to place the blue pod in blue node, red pod in red node, etc.
  * Sharing the same kubernetes cluster with other teams so there are other pods in the cluster as well as other nodes.
    * We don't want any other pod to be placed on their nodes. 
    * Neither do we want our pods to be placed on their nodes
    * ![affinity-example-diagram](/images/affinity-example.jpg)
  
  
  * **Start by Trying to Solve with Taints and Tolerations**
    *  apply a taint to nodes, marking them with their colors
       *  blue, red, and green. 
    *  Set toleration on the pod to tolerate the respective colors when the parts are now created.
       *  Nodes ensure they only accept the pod with the right toleration. 
       *  Doesn't guarantee that nodes won't land in the other.
       *  ![Taint-Failed-Solution](/images/taint-failed-solution.jpg)
   
  *  **Try Now with Node Affinity**
      * Label Nodes with their respective colors, then set node selectors on the pods to tie the pod to the node.
        * This **doesn't** guarantee that other nodes won't land in our nodes though
        * ![Node-Affinity-Failed](/images/node-affinity-failed.jpg)  
        
    * **Taints/Tolerations and Node Affinity**
      * Can be used together to completely dedicate nodes for specific pods.  
        * First use taints and toleration to prevent other pods from being placed on our nodes.
        * Then node affinity to make sure that our pods get placed in the correct node.
        * ![Node-Taint](/images/node-affinity-failed.jpg)


## Resource Requirements and Limits
  
  
* Look at 3 Node Kubernetes cluster. 
 * Each node has a set of CPU, Memory and Disk resources available. 
 * Every pod consumes a set of resources
   * In this case 2 CPUs, one Memory and some disk space. 
   * Whenever a pod is placed on a Node, it consumes resources available to that node. 
* Kubernetes scheduler that decides which nodes a pod goes to, considers resources required by a POD 
* If scheduler has no sufficient resources, the scheduler avoids placing the POD on that node and instead places the POD on one where sufficient resources are available.
 * If no sufficient resources, kubernetes holds back scheduling the node and you will see the pod in a pending state. 
   * If you look at the events, you will see the reason - like `insufficient cpu`.
* By default, kubernetes assumes that a pod or container with a pod requires `.5 CPU` & `256Mb of memory`
  * Known as the **Resource Request** for a container. 
    * Minimum amount of CPU or memory requested by the container.
      * When the scheduler tries to place the pod on a node, uses these numbers to identify a node that has sufficient amount of resources available.
      * If you know your app will need more, you can specify them in pod or deployment definition file. 
         `pod-defintion.yaml`
         ```yaml
         apiVersion: v1
         kind: Pod
         metadata:
          name: simple-webapp-color
          labels:
            name: simple-webapp-color
        spec:
          containers:
          - name: simple-webapp-color
            image: simple-webapp-color
            ports:
              - containerPort: 8080
            ### Resources Thing 
            resources:
              requests:
                memory: "1Gi" #Gi instead of GB because they are hipsters i guess
                #set to 1gb of memory and 1 count of vCPU
                cpu: 1 
            ##################
         ```
        * What does one count of CPU mean? 
          * Can specify any value as low as `0.1`
            * Can also be expressed as `100m`
            * can go as low as `1m` but not lower than that. 
          * 1 count of CPU is equivalent to 1 vCPU. 
            * 1 AWS vCPU
            * 1 GCP Core
            * 1 Azure Core
            * 1 Hyperthread
        * Can request a higher number of CPUs for the container, provided your nodes are sufficiently funded.
          * 1 G (Gigabyte)  = 1,000,000,000 bytes
          * 1 M (Megabyte)  = 1,000,000 bytes
          * 1 K (Kilobyte)  = 1,000 bytes
          
          * 1 Gi (Gibibyte) = 1,073,741,824 bytes
          * 1 Mi (Mebibyte) = 1,048,576 bytes
          * 1 Ki (Kibibyte) = 1,024 bytes  

### Resource Limits

* Look at container running on a node
  * In the docker world, a docker container has no limit to the amount of resources it can consume.
  * Say a container starts with 1 vCPU on a Node, it can go up and consume as much resources as it requires
    * Suffocates the native processes on the node or other containers
    * Can set a limit for the resource usage on these pods by default. 

* By default, kubernetes sets a limit of 1vCPU to containers. 
  * if you don't specify explicitly, a container will be limited to consume only one vCPU on a node. 
  * Same goes with memory. Kubernetes sets a limit of **512 Mi** on containers
* If you don't like default limits, you can change them by adding a limit section under the resources 
  ```yaml
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
     #set limits here
    limits:
      memory: "2Gi"
      cpu: 2
  ```
  * When pod is created, kubernetes sets new limits for the container
    * **LIMITS ARE SET PER CONTAINER, NOT PER POD**
* What happens when a pod tries to exceed resources beyond specified limit?
  * CPU
    * Kubernetes **throttles CPU** so it doesn't go beyond specified limits
      * A container can't use more CPU than it's limit
  * Memory
    * Container can use more memory resources than its limit. 
    * If a pod tries to consume more memory than it's limit constantly, pod is **Terminated**

## Daemon Sets

* Daemon sets are like replica sets. 
  * helps you deploy multiple instance of your pod. 
  * Runs one copy of your pod on each node in your cluster. 
  * Whenever a new node is added to the cluster, a replica is automatically added to the node. 
    * When a node is removed the pod is automatically removed. 
* **Daemon set ensures that one copy of the pod is always present in all nodes in the cluster.**

### Use Case
  * Say you would like to deploy a monitoring agent or log collector on each of your nodes in the cluster to monitor your cluster better.
    * Daemon set is prefect for that since it can deploy your monitoring agent in the form of a pod in all the nodes in your cluster
    * don't have to worry about adding or removing monitoring nodes in your cluster. 
    * ![daemon-set-use-case](/images/Daemon-Set-Use.jpg)

  * **Kube-Proxy Use Case** 
    * We know that kube-proxy is required on every node in the cluster. 
    * kube-proxy component can be deployed as the daemon set in cluster
  
  * **Networking**
    * Weave-net is a good use case since it's required on every node in the cluster. 

### Creation
* Similar to replicaset-definition
`daemon-set-definition.yaml`
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:

```









# Quick Notes

## Editing Pods and Deployments
  
  ### Edit a POD

  * You cannot edit specifications of an existing pod _other_ than these
    * `spec.containers[*].image`
    * `spec.initContainers[*].image`
    * `spec.activeDeadlineSeconds`
    * `spec.tolerations`
  * You cannot edit the environment variables, service accounts, resource limits, of a running pod. if you really want to though, there are two options
    1. `kubectl edit pod <pod name>` 
      * will open the pod specification in vim, then edit the required properties. When you try to save it, you will be denied. this is because you are attempting to edit a field on the pod that is not editable. 
      * A copy of the file with your changes is saved in a tempory location as shown above.
      * `kubectl delete pod <pod name>`
      * Then create a new pod with your changes using the temporary file
        * `kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`
    2. Extract the pod definition in YAML format
       * `kubectl get pod webapp -o yaml > my-new-pod.yaml`
       * Make changes to exported file using an editor
         * `vim my-new-pod.yaml`
       * Then delete existing pod
         * `kubectl delete pod webapp`
       * then create a new pod with the edited file
         * `kubectl create -f my-new-pod.yaml`  
  ### Edit Deployments
    * You can easily edit any field/property of the pod template
      * Since the pod template is a child of the deployment specification, with every change the new deployment will automatically delete and create a new pod with the new changes. 
      * If you are asked to edit a property of a POD part of a deployment can simply run this command.
        * `kubectl edit deployment my-deployment`









# End Table of Contents
1. [Table of Contents](#table-of-contents)
2. [Core Concepts](#core-concepts)
   1. [Cluster Architecture](#cluster-architecture)
   2. [ETCD](#etcd)
      1. [ETCD For Beginners](#etcd-for-beginners)
      2. [ETCD in Kubernetes](#etcd-in-kubernetes)
   3. [Kube-API Server](#kube-api-server)
   4. [Kube Controller Manager](#kube-controller-manager)
   5. [Kube Scheduler](#kube-scheduler)
   6. [Kubelet](#kubelet)
   7. [Kube Proxy](#kube-proxy)
   8. [PODs](#pods)
      1. [Recap](#recap)
      2. [How to deploy Pods](#how-to-deploy-pods)
      3. [PODs with YAML](#pods-with-yaml)
   9. [ReplicaSets](#replicasets)
      1. [Creating a Replication Controller](#creating-a-replication-controller)
      2. [Creating a ReplicaSet](#creating-a-replicaset)
      3. [Labels and Selectors](#labels-and-selectors)
   10. [Deployments](#deployments)
      1. [What is a Deployment](#what-is-a-deployment)
      2. [Definition](#definition)
   11. [Helpful Pod Commands](#helpful-pod-commands)
   12. [Namespaces](#namespaces)
      1. [Basics](#basics)
      2. [Resource Limits](#resource-limits)
      3. [DNS](#dns)
      4. [Commands](#commands)
   13. [Services](#services)
      1. [Services Use Case](#services-use-case)
      2. [Service Types - Basics](#service-types---basics)
      3. [NodePort](#nodeport)
      4. [Cluster IP](#cluster-ip)
         1. [Creation](#creation)
   14. [Imperative Commands](#imperative-commands)
      1. [POD](#pod)
      2. [Deployment](#deployment)
      3. [Service](#service)
      4. [Misc.](#misc)
3. [Scheduling](#scheduling)
   1. [Manual Scheduling](#manual-scheduling)
      1. [Labels and Selectors](#labels-and-selectors-1)
      2. [Creating Labels](#creating-labels)
      3. [Selectors](#selectors)
      4. [Annotations](#annotations)
   2. [Taints and Tolerations](#taints-and-tolerations)
      1. [Commands](#commands-1)
      2. [Master Nodes](#master-nodes)
   3. [Node Selectors](#node-selectors)
      1. [Limitations](#limitations)
   4. [Node Affinity](#node-affinity)
      1. [Node Affinity Types](#node-affinity-types)
   5. [Node Affinity Vs Taints and Tolerations](#node-affinity-vs-taints-and-tolerations)
   6. [Resource Requirements and Limits](#resource-requirements-and-limits)
      1. [Resource Limits](#resource-limits-1)
   7. [Daemon Sets](#daemon-sets)
      1. [Use Case](#use-case)
      2. [Creation](#creation-1)
4. [Quick Notes](#quick-notes)
   1. [Editing Pods and Deployments](#editing-pods-and-deployments)
      1. [Edit a POD](#edit-a-pod)
      2. [Edit Deployments](#edit-deployments)
5. [End Table of Contents](#end-table-of-contents)
