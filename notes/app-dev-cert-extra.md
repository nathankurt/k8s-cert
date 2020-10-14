# Table of Contents
- [Table of Contents](#table-of-contents)
- [Configuration](#configuration)
  - [Command Arguments in Kubernetes](#command-arguments-in-kubernetes)
  - [Editing PODs and Deployments](#editing-pods-and-deployments)
    - [Edit a POD](#edit-a-pod)
    - [Edit Deployments](#edit-deployments)
  - [Environment Variables in Kubernetes](#environment-variables-in-kubernetes)
  - [ConfigMaps](#configmaps)
    - [ConfigMap in Pods](#configmap-in-pods)
  - [ServiceAccounts for CKAD](#serviceaccounts-for-ckad)
  - [Resource Requirements CKAD](#resource-requirements-ckad)
- [Multi-Container PODs](#multi-container-pods)
  - [Sidecar](#sidecar)
  - [Adapter](#adapter)
  - [Ambassador](#ambassador)
- [Observability](#observability)
  - [Readiness and Liveness Probes](#readiness-and-liveness-probes)
    - [Readiness Probes](#readiness-probes)
    - [HTTP Test - /api/ready](#http-test---apiready)
    - [TCP Test - 3306](#tcp-test---3306)
    - [Exec Command](#exec-command)
  - [Liveness Probe](#liveness-probe)
    - [HTTP Test - /api/ready](#http-test---apiready-1)
    - [TCP Test - 3306](#tcp-test---3306-1)
    - [Exec Command](#exec-command-1)
  - [Container Logging](#container-logging)
- [POD Design](#pod-design)
  - [Jobs](#jobs)
    - [Parallelism](#parallelism)
  - [Cron Jobs](#cron-jobs)
- [Services and Networking](#services-and-networking)
  - [Services](#services)
    - [Services Use Case](#services-use-case)
    - [Service Types - Basics](#service-types---basics)
    - [NodePort](#nodeport)
    - [Cluster IP](#cluster-ip)
      - [Creation](#creation)
  - [Network Policies](#network-policies)


# Configuration

## Command Arguments in Kubernetes

* Created a simple docker image that sleeps for a given number of seconds before named `ubuntu-sleeper`
  * Ran by running command `docker run --name ubuntu-sleeper ubuntu-sleeper`
  * By default it sleeps for five seconds but you can override it by passing a command line argument
    * `docker run --name ubuntu-sleeper ubuntu-sleeper 10`

* If you wanted to override the entrypoint option in docker you would run the command with the `entrypoint` option set to the new command
  * `docker run --name ubuntu-sleeper --entrypoint sleep2.0 ubuntu-sleeper 10`
* Now Create a pod using this image

```yaml
apiVersion:
kind: Pod
metadata:
    name: ubuntu-sleeper-pod
spec:
    containers:
      - name: ubuntu-sleeper
        image: ubuntu-sleeper

        ################## 
        #If you want to change entrypoint use command
        command: ["sleep2.0"]
        # If you want to change seconds use this arg
        args: ["10"]
        # Override CMD instruction in docker file        
        ################### 

```

* `args` = Docker CMD 
* `command` = Docker ENTRYPOINT


## Editing PODs and Deployments
A quick note on editing PODs and Deployments

### Edit a POD

Remember, you CANNOT edit specifications of an existing POD other than the below.

  *  spec.containers[*].image

  *  spec.initContainers[*].image

  *  spec.activeDeadlineSeconds

  *  spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`


Then create a new pod with your changes using the temporary file

`kubectl create -f /tmp/kubectl-edit-ccvrq.yaml`


2. The second option is to extract the pod definition in YAML format to a file using the command

`kubectl get pod webapp -o yaml > my-new-pod.yaml`

Then make the changes to the exported file using an editor (vi editor). Save the changes

`vi my-new-pod.yaml`

Then delete the existing pod

`kubectl delete pod webapp`

Then create a new pod with the edited file

`kubectl create -f my-new-pod.yaml`


### Edit Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

`kubectl edit deployment my-deployment `


## Environment Variables in Kubernetes 

* To set an environment variable, use the `env` property

`docker run -e APP_COLOR=pink simple-webapp-color`

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
###############
    env: 
      - name: APP_COLOR
        value: pink
##############
```

* This is the Plain Key Value Pair way of doing things
* Can also set env variables with things like 

** ConfigMap
```yaml
env: 
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

** Secrets
```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

## ConfigMaps

* When you have a lot of pod definition files, will be tough to manage data. 
* Can take info out of pod using config maps 

* Pass config data in the form of key value pairs

* Can create it imperatively `kubectl create configmap`
   
  * `kubectl create configmap <config-name> --from-literal=<key>=<value>`

  *  `kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod` 
* Can Also create it through file
  * `kubectl create configmap <config-name> --from-file=<path-to-file>`
  * `kubectl create configmap app-config --from-file=app_config.properties`

* Declaritive way with `kubectl create -f`

    ```yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: config-map
      namespace: default
    data:
      key: value
      key2: value2
    ```

* To view config maps `kubectl get configmaps`
  * describe `kubectl describe configmaps`

### ConfigMap in Pods

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
    - containerPort:  8080
    envFrom: 
      - configMapRef:
          name: app-config
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-config
data:
  APP_COLOR: pink
  APP_MODE: prod
```


## ServiceAccounts for CKAD

* To create serviceaccount run `kubectl create serviceaccount <SA-Name>`
* To view the serviceaccount `kubectl get serviceacccount`
  * When serviceaccount is created, it also creates a token automatically
    * `kubectl describe serviceaccount dashboard-sa`
  * Token is what must be used by the external app while authenticating to the kubernetes API 
  * Token is stored as a secret object 


* When serviceaccount is created, first creates the service account object
  * Then generates a token for service account
    * Then creates a secret object and stores that token inside the secret object.
    * Secret object is then linked to the service account
    * To view seccret view of token run `kubectl describe secret dashboard-sa-token-kbbdm`
    * This token can then be used as an authentication bearer toekn while making your REST call to the kubernetes API 

* What if third party app is hosted o n the kubernetes cluster itself?
  * Can automatically be done by mounting the service token secret as a volume inside the pod hosting the third party application
  * Don't have to provide it manually, if you go back and loo at the list of service accounts, you will see there is a default service account that exists already. For every namespace in Kubernetes 

* Whenever a pod is created, the default service account and it's token are automatically mounted to that pod as a volume mount.
  * For example we have a simple pod definition file that creates a pod using my custom kubernetes dashboard image.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-kubernetes-dashboard
  spec:
    containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
      
  ```
  * Haven't specified any secrets or Volume mounts in the definition file, however when the pod is created you look at the details of the pod by running `kubectl describe pod` and see that a volume is automatically created from the secret named "default-token" 
    * aka the secret containing the token for this default service account
    * secret token is mounted at location `/var/run/secrets/kubernetes.io/service/account` inside the pod 
    * If you run the command `kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount` you get three seperate files. 
      * The one with the actual token is the file named "token"

  * If you'd like to use a different service account add `serviceAccount` field to YAML
  * **CANNOT** Edit the service account of an existing pod, in the case of a deployment though, you will be able to edit the service account because deployment will take care of deleting and recreating new accounts. 
  * Can set automount to true by setting `automountServiceAccountToken: False` in pod spec section


## Resource Requirements CKAD 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "MYAPP"
  namespace: default
  labels:
    app: "MYAPP"
spec:
  containers:
  - name: MYAPP
    image: "debian-slim:latest"
    resources:
      limits:
        memory: "2Gi"
        cpu: 1
      requests:
        cpu: 1
        memory: "1Gi"
```

Min amount of CPU or memory requested by the container when the scheduler tries to place the pod on the node. 

* 1 count of CPU = 
  * 1 AWS vCPU
  * 1 GCP Core
  * 1 Azure Core
  * 1 Hyperthread

* Can also lset a limit so it doesn't sufficate native processes on the nodes or other containers 
* Set for each container within the pod 
* If it exceeds CPU it throttles, with memory it will terminate the pod. 

Note on default resource requirements and limits

In the previous lecture, I said - "When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

  ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: mem-limit-range
    spec:
      limits:
      - default:
          memory: 512Mi
        defaultRequest:
          memory: 256Mi
        type: Container
  ```
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-limit-range
    spec:
      limits:
      - default:
          cpu: 1
        defaultRequest:
          cpu: 0.5
        type: Container
    ```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/


References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource


# Multi-Container PODs

* Don't want to merge and bloat the code of the two services as each of them target different functionalities and would still like them to be developed and deployed separately
* Only need the two functionality to work together
  * Need one agent per web server instance paired together that can scale up and down together 
  * Have multi container pods that share the same lifecycle, network, and storage 

* All you need to do is add the new container of information to the pod definition file. Remember the container section under 

## Sidecar
Just like the logging service
* Deploying a logging agent alongside a web server to collect logs and forward them to a central log server 

## Adapter
* Say we have multiple applications generating logs in different formats. Would be hard to process the various formats on the central logging server. 
* So before sending the logs to the central server, we would like to convert the logs to a common format
  * Deploy an Adapter container processes logs before sending it to central server


## Ambassador

* App communicates to different database instances at different stages of development.
* Local database for dev, one for testing, another for prod
* Must make sure to modify this connectivity in your app code depending on the environment you are deploying the app to. 
  * May choose to outsource such logic to a separate container within your pod so that your application can always refer to a database at a local host and the new container will proxy that request to the right database.


# Observability

## Readiness and Liveness Probes

### Readiness Probes

* Conditions compliment pod status 
  * It is an array of true or false values that tell us the state of a pod when a pod is schedule on a node 
  * Run `kubectl describe pod` and look for conditions section
  * ![pod-conditions](/images/pod-conditions.jpg)
* Ready condition indicates that the application inside the pod is running and is ready to accept user traffic

* By default, kubernetes assumes that as soon as teh container is created it is ready to server user traffic.
  * So it sets the value of the ready condition for each container to true
  * But if the application within the container took longer to get ready, service is unaware and sends traffic through as the container is already in a ready state causing users to hit a POD that isn't yet running a live web application
* Need a way to tie a ready condition to the actual app in the container. 
  * Set up tests/probes Test when the API server is up and running
  * Can see if a particular TCP socket is listening 
  * Or an exec command that would exit successfully if app is running

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "MYAPP"
  namespace: default
  labels:
    app: "MYAPP"
spec:
  containers:
  - name: MYAPP
    image: "debian-slim:latest"
    ports:
    - containerPort:  8080
      name:  http
    ############################  
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
    ##########################
```

* Now when container is created, kub does not immediately set the ready condition on the container to true
  * performs a test to see if the API responds positively 

Different ways for each

### HTTP Test - /api/ready
```yaml
readinessProbe:
  httpGet:
    path: /api/ready
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

### TCP Test - 3306
```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```
### Exec Command
```yaml
readinessProbe:
  exec
    command:
      - cat
      - /app/is_ready
```


## Liveness Probe
* Every time an app crashes, kub makes an attempt to restart the container to restore service
  * Can see count of restarts increase in `kubectl get pods`
* What if the app is not really working but the container continues to stay alive? 
  * Say for example due to a bug in the code the application is stuck in an infinite loop
    * As far as kube is concerned,app is up, container is up so app is assumed to be up. 
* This is where liveness probes come in. 
   
   

### HTTP Test - /api/ready
```yaml
livenessProbe:
  httpGet:
    path: /api/ready
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

### TCP Test - 3306
```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```
### Exec Command
```yaml
livenessProbe:
  exec
    command:
      - cat
      - /app/is_ready
```


## Container Logging

* Once pod is running, we can view the logs using `kubectl logs <POD-NAME>` command `-f` streams the logs live
* If pod has multiple containers, must also specify the name of the container, otherwise it will fail and ask you to sepcify a name 



# POD Design

**See K8s adm notes for Selectors and deployment rollbacks**

## Jobs 

* Other kinds of workloads such as batch processing analytics or reporting that are meant to carry out a specific task and then finish. 

* Create a pod to add two numbers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
  - name: math-add
    image: ubuntu
    command: ['expr', '3', '+', '2']
  ############# Default set to always so keeps running
  restartPolicy: Always
  ############# Can override to Never or OnFailure
```

* It runs the task and exits and the pod goes into a completed state, but it then recreates the container in an attempt to leave it running. 
  * Should go into loop

* Now we have large data sets that require multiple PODs to process data in parallel. 
* Want all PODs perform the task assigned to them successfully and then exit. 
**Kubernetes Job**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

* `kubectl get jobs`
* Output of the job? 
  * Standard output of container can be used with `kubectl logs math-add-job-1d87pm`
* Delete job will delete pods as well
  * `kubectl delete job blah-blah`


### Parallelism

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

## Cron Jobs
* Job that can be scheduled 

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-jb
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
    completions: 3
    parallelism: 3
    template:
      spec:
        containers:
          - name: reporting-tool
            image: reporting-tool
            command: ['expr', '3', '+', '2']
        restartPolicy: Never
```

* `kubectl get cronjob`


# Services and Networking

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

## Network Policies
* Use label and selectors for network policies
* Then build rule
```yaml
apiVersion: networking.k8s.io/v1
kind: 
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            name: api-pod
      ports:
        protocol: TCP
        port: 3306
```
  
  
    
    