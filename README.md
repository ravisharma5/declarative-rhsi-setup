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
Above will restart the terminal window on the OpenShift console and set the image which has Skupper pre-installed on it. Feel free to update the version of it if required.

## Deploy Applications to your clusters
 Deploy Frontend app to cluster A
```yaml
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

oc apply -f frontend.yaml

 Deploy Backend app to cluster B

 


## Create Skupper sites with a config map

## Create Token and link sites

## Exposing services to the service network created by Skupper
