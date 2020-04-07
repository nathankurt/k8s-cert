- [Definitions](#definitions)
  - [POD](#pod)
    - [Namespace In Pod](#namespace-in-pod)
  - [ReplicaSet](#replicaset)
  - [ReplicationController](#replicationcontroller)
  - [Deployments](#deployments)
  - [NameSpace](#namespace)
  - [Resource Quota](#resource-quota)
  - [NodePort Service](#nodeport-service)
  - [ClusterIP Service](#clusterip-service)
  - [Binding Object](#binding-object)
  - [Selectors](#selectors)
  - [Annotations](#annotations)
  - [Toleration](#toleration)
  - [Node Selectors](#node-selectors)
  - [Node Affinity](#node-affinity)
  - [Resource Requirements](#resource-requirements)
  - [Resource Limits](#resource-limits)
  - [Daemon Sets](#daemon-sets)
  - [Deploy Additional Scheduler - -kubeadm](#deploy-additional-scheduler----kubeadm)
    - [Kube Scheduler](#kube-scheduler)
    - [Custom Scheduler](#custom-scheduler)
    - [Use Custom Scheduler](#use-custom-scheduler)
  - [Event-Simulator](#event-simulator)
  - [Docker Args in Pod](#docker-args-in-pod)
  - [Environment Variables](#environment-variables)
    - [Plain Key Values](#plain-key-values)
    - [ConfigMaps](#configmaps)
    - [Secrets](#secrets)
  - [Multi Container Pods](#multi-container-pods)
  - [InitContainers](#initcontainers)
  - [Cert Config](#cert-config)
  - [OpenSSL Config](#openssl-config)
  - [Kubelet-Config](#kubelet-config)
  - [CertifcateSigningRequest](#certifcatesigningrequest)
  - [KubeConfig](#kubeconfig)
  - [Role Based Access Controls](#role-based-access-controls)
    - [Role](#role)
    - [RoleBinding](#rolebinding)
  - [Cluster Roles and Role Bindings](#cluster-roles-and-role-bindings)
    - [Role](#role-1)
    - [RoleBinding](#rolebinding-1)
  - [Volumes and Mounts](#volumes-and-mounts)
    - [Persistent Volumes](#persistent-volumes)
    - [Persistent Volume Claims](#persistent-volume-claims)
    - [Persistent Volume Claims in PODS](#persistent-volume-claims-in-pods)
- [Networking](#networking)
  - [Ingress](#ingress)
    - [Ingress Controller](#ingress-controller)
      - [Config Maps](#config-maps)
      - [Service Account](#service-account)
    - [Ingress Resources](#ingress-resources)
    - [Annotations and Rewrite Targets](#annotations-and-rewrite-targets)

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


## CertifcateSigningRequest

`jane-csr.yaml`
```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  ##List the groups the user should be part of
  groups:
  - system:authenticated
  ## And usages of the account as a list of strings 
  usages:
  - digital signature
  - key encipherment
  - server auth
  # Don't specify request as plain text, instead encode it with 
  # base64 command Then move encoded text into the request 
  # field and submit the request once object is created
  # cat jane.csr | base64
  request:
  # cat jane.csr | base64 output

``` 

More Info [here](/notes/core-concepts.md/#certificates-api)

## KubeConfig

`config` or `my-config.yaml` but config is the default where you don't have to use a create
```yaml
apiVersion: v1
kind: Config
#Selects the default context to use
current-context: my-kube-admin@my-kube-playground
#Each of these things is in an array format 
#so you can specify multiple clusters
clusters:
- name: production
  cluster:
    certificate-authority: ca.crt
    server: https://172.17.0.51:6443


contexts:
- name: admin@production
  context: 
    cluster: production #name of cluster above
    user: admin #name of user below
    # can add namespace here as well. 
    namespace: finance

users:
- name: admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

More Info [here](/notes/core-concepts.md/#file-format)

## Role Based Access Controls

### Role

`developer-role.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
  #You can select the names of the pods people have access to
  resourceNames: ["blue", "orange"]
  
  #Rule to allow developers to create config maps
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]    
```

### RoleBinding

`devuser-developer-binding.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
#Where we specify the user details
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
#where we provide the details of the role we created
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

More Info [here](/notes/core-concepts.md/#role-based-access-controls)


## Cluster Roles and Role Bindings


### Role

`cluster-admin-role.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
``` 

### RoleBinding

`cluster-admin-role-binding.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

More Info [here](/notes/core-concepts.md/#cluster-roles-and-role-bindings)

## Volumes and Mounts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"] 
    #################################################################
    #After volume is created, to access it from a container we 
    #mount the volume to a directory inside the container
    volumeMounts:
      #now random number will be written to /opt
      #inside the container which happens the data volume
      #which is infact the data directory on the host
      #when pod is delete, file still lives on the host
      - mountPath: /opt
        name: data-volume 
  
    #To retain the random number, you need this volume
  volumes:
  - name: data-volume
    #configure storage to use a directory on the host 
    #any files created in the volume would be stored in the 
    #directory dtta on my node
    hostPath:
      path: /data
      type: Directory
  ######################################################################  
```

More Info [here](/notes/core-concepts.md/#volumes)

### Persistent Volumes

`pv-definition.yaml`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv-vol1

spec:
#################################
  accessModes:

    - ReadWriteOnce
  #- ReadWriteMany
  #- ReadOnlyMany

  capacity:
    storage: 1Gi

  #don't use this option in a prod environment
  hostPath:
    path: /tmp/data
######################################
```

Can also replace host path options with supported storage solutions:

```yaml
awsElasticBlockStore:
  volumeID: <volume-id>
  fsType: ext4
```

More Info [here](/notes/core-concepts.md/#persistent-volumes)

### Persistent Volume Claims

`pvc-definition.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi

```
More Info [here](/notes/core-concepts.md/#persistent-volume-claims)

### Persistent Volume Claims in PODS

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      ###########################
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypod
  volumes:
    - name: mypod
      persistentVolumeClaim:
        claimName: myclaim
#########################################
```

More Info [here](/notes/core-concepts.md/#using-persistent-volume-claims-in-pods)

# Networking

## Ingress 

Similar to a service, but more like a load balancer built in to the kubernetes cluster.

### Ingress Controller

`nginx-ingress-controller.yaml`
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          ##Special build of nginx built specifically to be used as an ingress controller in kubernets

      args: 
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace  

      #specify the ports used by the ingress controller(happens to be 80 and 443)
      ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

More Info [here](/notes/core-concepts.md/#ingress-controller)

#### Config Maps

Decouple config data from nginx-controller image, you must create a ConfigMap object and pass that in

`nginx-config.yaml`
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```

More Info [here](/notes/core-concepts.md/#configmaps)

#### Service Account
 
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount 
```

More Info [here](/notes/core-concepts.md/#serviceaccounts)

### Ingress Resources

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  #application is routed to applicaion services and not pods directly
  backend: #defines where traffic will be routed to
    serviceName: wear-service
    servicePort: 80
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  # requirement is to handle all traffic coming to `my-online-store.com`
  # route them based on the URL path
  # just need a single url path. 
  rules:
  - http:
      paths:

      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80 

      - path: /watch
        backend: 
          serviceName: watch-service
          servicePort: 80 
``` 


`Ingress-wear-watch.yaml`
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  
  rules:

  - host: wear.my-online-store.com
    http:
      paths:
        - backend:
            serviceName: wear-service
            servicePort: 80
    #if you odn't specify the host field, it will consider it as a star.         
  - host: watch.my-online-store.com
    http:
      paths:
        - backend:
            serviceName: watch-service
            servicePort: 80  

```

Learn More [here](/notes/core-concepts.md/#ingress---annotations-and-rewrite-target)

### Annotations and Rewrite Targets

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
        - path: /pay
          backend:
            serviceName: pay-service
            servicePort: 8282  
  ```


In another example given [here](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:
  `replace("/something(/|$)(.*)", "/$2")`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
  
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
        serviceName: http-svc
        servicePort: 80
      path: /something(/|$)(.*)  
```
