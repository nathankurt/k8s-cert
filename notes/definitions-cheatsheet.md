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

