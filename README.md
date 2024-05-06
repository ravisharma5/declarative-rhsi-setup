# declarative-rhsi-setup
Repo with instructions to set Red Hat Service Interconnect and manage the sites declaratively

## Setup Red Hat Service Interconnect(RHSI) on OpenShift using operator framework
Install the latest RHSI operator.
```bash
oc apply -f subscription-rhsi.yaml
```

#### Optional component if you don't want to install Skupper cli on your machine.
Setup Webterminal to help watch the skupper pods and generate required tokens for connecting other sites
```bash
oc apply -f subscription-web-terminal.yaml
```
If Skupper is not installed as part of your Webterminal toolkit then set the image to the below image within the terminal window.

```bash
wtoctl set image quay.io/redhatintegration/rhi-tools:dev3
```
Above will restart the terminal window on the OpenShift console and set the image that has Skupper pre-installed on it. Feel free to update the version of it if required.

## Deploy Applications to your clusters
#### Deploy Frontend app to cluster A. 
Below we will create a project/namespace, deployment, service, and a route to access the frontend app.

```bash
oc apply -f frontend.yaml
```
<details>

<summary>Contents of frontend.yaml </summary>

```yaml
---
apiVersion: project.openshift.io/v1
description: "frontend"
displayName: frontend
kind: ProjectRequest
metadata:
    name: frontend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  labels:
    app: frontend
  name: frontend
  namespace: frontend
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend
    spec:
      containers:
      - image: quay.io/skupper/hello-world-frontend
        imagePullPolicy: Always
        name: hello-world-frontend
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      terminationGracePeriodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: frontend
  name: frontend
  namespace: frontend
spec:
  clusterIP: 172.31.223.69
  clusterIPs:
  - 172.31.223.69
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 8080
  selector:
    app: frontend
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: frontend
  name: frontend
  namespace: frontend
spec:
  port:
    targetPort: 8080
  to:
    kind: Service
    name: frontend
    weight: 100
  wildcardPolicy: None
```
</details>

#### Deploy Backend app to cluster B. 
Below we will create a project/namespace, deployment, service, and a route to access the backend app.

Deploy Backend app to cluster B
```bash
oc apply -f backend.yaml
```
<details>
    
<summary>Contents of backend.yaml </summary> 

```yaml
---
apiVersion: project.openshift.io/v1
description: "backend"
displayName: backend
kind: ProjectRequest
metadata:
    name: backend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  labels:
    app: backend
  name: backend
  namespace: backend
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: backend
    spec:
      containers:
      - image: quay.io/skupper/hello-world-backend
        imagePullPolicy: Always
        name: hello-world-backend
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      terminationGracePeriodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: backend
  name: backend
  namespace: backend
spec:
  clusterIP: 172.31.223.69
  clusterIPs:
  - 172.31.223.69
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 8080
  selector:
    app: backend
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: backend
  name: backend
  namespace: backend
spec:
  port:
    targetPort: 8080
  to:
    kind: Service
    name: backend
    weight: 100
  wildcardPolicy: None
```
</details>

## Create Skupper sites with a config map
Now we will enable skupper sites on both clusters
#### Apply Frontend skupper configmap to enable site
```bash
oc apply -f frontend-site.yaml
```
frontend-site.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: skupper-site
data:
  name: frontend-site
  console: "true"
  console-user: "admin"
  console-password: "openshift"
  flow-collector: "true"
```

Using webterminal or your own cli or via skupper console you can verify the site deployment. The skupper route will be available under the networking>route section of the console.

#### Apply Backend skupper configmap to enable site

```bash
oc apply -f backend-site.yaml
```
frontend-site.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: skupper-site
data:
  name: backend-site
  console: "true"
  console-user: "admin"
  console-password: "openshift"
  flow-collector: "true"
```

## Create Token and link sites

## Exposing services to the service network created by Skupper
