1. [Definitions](#definitions)
   1. [POD](#pod)
      1. [Namespace In Pod](#namespace-in-pod)
   2. [ReplicaSet](#replicaset)
   3. [ReplicationController](#replicationcontroller)
   4. [Deployments](#deployments)
   5. [NameSpace](#namespace)
   6. [Resource Quota](#resource-quota)
   7. [NodePort Service](#nodeport-service)
   8. [ClusterIP Service](#clusterip-service)
   9. [Binding Object](#binding-object)
   10. [Selectors](#selectors)
   11. [Annotations](#annotations)
   12. [Toleration](#toleration)
   13. [Node Selectors](#node-selectors)
   14. [Node Affinity](#node-affinity)
   15. [Resource Requirements](#resource-requirements)
   16. [Resource Limits](#resource-limits)
   17. [Daemon Sets](#daemon-sets)
   18. [Deploy Additional Scheduler - -kubeadm](#deploy-additional-scheduler----kubeadm)
      1. [Kube Scheduler](#kube-scheduler)
      2. [Custom Scheduler](#custom-scheduler)
      3. [Use Custom Scheduler](#use-custom-scheduler)
   19. [Event-Simulator](#event-simulator)
   20. [Docker Args in Pod](#docker-args-in-pod)
   21. [Environment Variables](#environment-variables)
      1. [Plain Key Values](#plain-key-values)
      2. [ConfigMaps](#configmaps)
      3. [Secrets](#secrets)
   22. [Multi Container Pods](#multi-container-pods)
   23. [InitContainers](#initcontainers)
   24. [Cert Config](#cert-config)
   25. [OpenSSL Config](#openssl-config)
   26. [Kubelet-Config](#kubelet-config)

# Definitions

## POD

```yaml
apiVersion: v1
kind: POD
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx

```
More Info [here](/notes/core-concepts.md/#pods-with-yaml)
  
  ### Namespace In Pod
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
More Info [here](/notes/core-concepts.md/#commands)

## ReplicaSet

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
More Info [here](/notes/core-concepts.md/#creating-a-replicaset)

## ReplicationController

`rc-definition.yml`
```yaml
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
More Info [here](/notes/core-concepts.md/#creating-a-replication-controller)

## Deployments

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
More Info [here](/notes/core-concepts.md/#definition)

## NameSpace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
* Can also just do `kubectl create namespace dev`

More Info [here](/notes/core-concepts.md/#create-namespace-like)
## Resource Quota

```yaml
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
More Info [here](/notes/core-concepts.md/#create-a-resource-quota)

## NodePort Service

```yaml
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
More Info [here](/notes/core-concepts.md/#nodeport)

## ClusterIP Service
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
More Info [here](/notes/core-concepts.md/#creation)

## Binding Object
```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target: #NEW STUFF
  apiVersion: v1
  kind: Node
  name: node02
```
More Info [here](/notes/core-concepts.md/#manual-scheduling)

## Selectors
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
More Info [here](/notes/core-concepts.md/#selectors)


## Annotations

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
More Info [here](/notes/core-concepts.md/#annotations)

## Toleration
```yaml
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
More Info [here](/notes/core-concepts.md/#commands-1)

## Node Selectors
```yaml
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
More Info [here](/notes/core-concepts.md/#node-selectors)


## Node Affinity 

```yaml
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

More Info [here](/notes/core-concepts.md/#node-affinity)

## Resource Requirements

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
More Info [here](/notes/core-concepts.md/#resource-requirements-and-limits)

## Resource Limits

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
More Info [here](/notes/core-concepts.md/#resource-limits)

## Daemon Sets
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent

```

More Info [here](/notes/core-concepts.md/#daemon-sets)


## Deploy Additional Scheduler - -kubeadm

### Kube Scheduler
* `/etc/kubernetes/manifests/kube-scheduler.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  # Command and associated options to start the scheduler
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    #used when you have multiple copies of the scheduler running on different master nodes.
    - --leader-elect=true
    ################################################################
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler   
``` 

### Custom Scheduler
`my-custom-scheduler.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  # Command and associated options to start the scheduler
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    #used when you have multiple copies of the scheduler running on different master nodes. 
    #picks which will lead scheduling activities when multiple copies of same scheduler are running
    - --leader-elect=true 
    ######################################### 
    - --scheduler-name=my-custom-scheduler
    #to differentiate custom scheduler from default during leader election process 
    - --lock-object-name=my-custom-scheduler  
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler   
``` 

More Info [here](/notes/core-concepts.md/#deploy-additional-scheduler---kubeadm)

### Use Custom Scheduler
`pod-definition.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
  schedulerName: my-custom-scheduler
```
More Info [here](/notes/core-concepts.md/#use-custom-scheduler)


## Event-Simulator

* Create pod `event-simulator.yaml`

```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
    
    - name: image-processor
      image: some-image-processor
```


More Info [here](/notes/core-concepts.md/#logs---kubernetes)


## Docker Args in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod

spec:

  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      ##Changes entrypoint command
      command: ["sleep2.0"]
      ## Anything that is appended to the docker run command will go into
      ## the "args" property of the pod definition in array form. 
      args: ["10"]
      ###############
``` 

More Info [here](/notes/core-concepts.md/#application-commands--arguments)


## Environment Variables

### Plain Key Values
  * Plain Key Value:
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
    ### SET ENV VARIABLE
    env: #array so each thing starts with a dash. 
      - name: APP_COLOR
        value: pink

    ###############
```


### ConfigMaps

`config-map.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data: #instead of spec
  APP_COLOR: blue
  APP_MODE: prod
``` 

`pod-definition.yaml`
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
    ### SET Config Map to Pod
    envFrom: 
      - configMapRef:
          name: app-config
          ## The name from the config-map.yaml file
    
```

* Can inject it as a single environment variable

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

* Can inject whole data as files in a volume.

```yaml
volumes:
- name: app-config-volume
  configMap:
    name: app-config
```

More Info [here](/notes/core-concepts.md/#create-the-configmaps)

### Secrets

* `secret-data.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  #MUST SPECIFY SECRET VALUES IN HASHED FORMAT
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
  ####
```

* Secret in Pod `pod-definition.yaml`
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
    ### SET Secretp to Pod
    envFrom: 
      - secretRef:
          name: app-secret
          ## The name from the secret.yaml file 
```

* Can inject it as a single environment variable

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Password
```

* Can inject whole data as files in a volume.

```yaml
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

More Info [here](/notes/core-concepts.md/#secrets)


## Multi Container Pods

`pod-definition.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
    ### Since value is an array, you can add another container here
    - name: log-agent
      image: log-agent
    ###########
```

More Info [here](/notes/core-concepts.md/#create-multi-container-pod)

## InitContainers

`pod-definition.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

More Info [here](/notes/core-concepts.md/#initcontainers)

## Cert Config

`kube-config.yaml`
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
    server: https://kube-apiserver:6443
  name: kubernetes
  kind: Config
  users:
  - name: kubernetes-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
```

More Info [here](/notes/core-concepts.md/#what-to-do)

## OpenSSL Config

`openssl.cnf`
```conf
[req]
req_extensions = v3_req
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 172.17.0.87
```

More Info [here](/notes/core-concepts.md/#server-certificate-creation)

## Kubelet-Config 

`kubelet-config.yaml (node01)`
```yaml
kind: kubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/node01.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/node01.key" 
```


More Info [here](/notes/core-concepts.md/#server-certificate-creation)


