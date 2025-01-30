# OpenShift-Labs
Documentation and experience from my OpenShift labs.


## Overview
This repository documents my experience and learnings from completing the OpenShift lab as part of my studies at **Alnafi**. At Alnafi, I am gaining hands-on experience in cloud technologies and container orchestration, which are critical skills in today's tech 🌏.
<br>
<br>
# Lab 1: Managing OpenShift Container Platform

## Lab Objectives
- Understand and use the OpenShift web console.
- Use the CLI for cluster management.
- Create and manage projects in OpenShift.

## Tasks Completed

### 1. Log in to the OpenShift Web Console
- **Description**: Successfully logged in and explored the Overview section.
- **Key Learnings**: Understanding cluster health and resource usage.

### 2. Use the CLI to Log in to an OpenShift Cluster
- **Description**: Used the `oc login` command to access the cluster.
- **Key Learnings**: Familiarity with command-line operations.

### 3. Create a New Project Using the CLI
- **Command**:
  ```bash
  oc new-project myproject
  ```
- **Description**: Created a new project for my application.
- **Key Learnings**: Importance of organizing resources.

### 4. List and Delete Projects Using the CLI
- **Commands**:
```bash
oc get projects
oc delete project myproject
```
- **Description**: Managed projects via the CLI.
- **Key Learnings**: Resource management and cleanup.

## Conclusion for Lab 1
This lab enhanced my understanding of managing OpenShift clusters.

---
<br>
<br>

# Lab 2: Deploying Applications in OpenShift

### **Lab Objectives**
- Deploy applications using container images.
- Use OpenShift templates for deployment.
- Scale and manage deployments.
- Expose applications for external access.

### **Tasks Completed**

#### **1. Deploy an Application from a Container Image**
- **Command:** `oc new-app nginx:latest`
- **Description:** Deployed an NGINX web server using OpenShift CLI.
- **Key Learnings:** How to create a deployment from a container image.

#### **2. Deploy an Application Using an OpenShift Template**
- **Command:** `oc new-app -f nginx-template.yaml`
- **Description:** Used a template to deploy an NGINX web server.
- **Key Learnings:** How to define and apply templates for reusable configurations.

#### **3. Scale the Application**
- **Command:** `oc scale dc nginx --replicas=3`
- **Description:** Increased the number of running pods to 3.
- **Key Learnings:** How to scale applications based on demand.

#### **4. Work with Replica Sets**
- **Command:** `oc get rs`
- **Description:** Checked how replica sets maintain the desired number of pods.
- **Key Learnings:** Understanding automatic pod management.

#### **5. Expose the Application to External Users**
- **Command:** `oc expose service nginx`
- **Description:** Created a route to make the application accessible.
- **Key Learnings:** How to expose applications using OpenShift routes.

### **Conclusion for Lab 2**
This lab helped in deploying, managing, and exposing applications in OpenShift. It provided hands-on experience with scaling, templates, and routing.
---
<br>
<br>

# Lab 3: Managing Storage

## Objectives

- Store and retrieve sensitive information securely.
- Provision persistent storage for applications.
- Use storage classes and manage stateful applications.

## Tasks Completed

### 1. Create and Use Secrets for Sensitive Data

#### Description

Created a secret to store sensitive information and used it in a pod.

#### Commands

```bash
oc create secret generic passwd --from-literal=password=mySecret123
```
```bash
oc apply -f secret.yml
```
#### YAML Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: secret-container
      image: nginx  
      env:
        - name: passwd
          valueFrom:
            secretKeyRef:
              name: passwd
              key: password
```

#### Key Learnings

- Secrets help store sensitive data securely.
- Environment variables can reference secrets dynamically.

---

### 2. Create and Use ConfigMaps for Configuration Data

#### Description

Created a ConfigMap and used it in a pod to store application configuration.

#### Commands

```bash
oc create configmap my-config --from-literal=app-config=settingValue
```
```bash
oc apply -f config.yml
```
#### YAML Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-map-pod
spec:
  containers:
    - name: config-map-container
      image: nginx  
      env:
        - name: CONFIG_KEY
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: CONFIG_KEY
```

#### Key Learnings

- ConfigMaps store non-sensitive configuration data.
- Configuration can be managed separately from application code.

---

### 3. Provision Persistent Storage and Attach It to a Pod

#### Description

Created a PersistentVolume (PV) and PersistentVolumeClaim (PVC) and used them in a pod.

#### YAML Definitions

**PersistentVolume:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi 
  accessModes:
    - ReadWriteOnce  
  azureDisk:
    diskName: mydisk 
    diskURI: /subscriptions/474dfc9c-44a1-41a4-8f21-c8090cbbe6be/resourceGroups/NetworkWatcherRG/providers/Microsoft.Compute/disks/mycluster-m7dzr-master-0_OSDisk
    kind: Managed

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc  
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
```bash
oc apply volume.yml
```

**Pod Using PVC:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
    - name: storage-container
      image: nginx  
      volumeMounts:
        - name: storage-volume
          mountPath: /usr/share/nginx/html  
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: my-pvc  
```
```bash
oc apply pvc_pod.yml
```

#### Key Learnings

- PVs provide persistent storage to pod.
- PVCs request storage from available PVs.

---

### 4. Understand and Use StorageClasses

#### Description

Defined a StorageClass to provision storage and used it in a PVC.

#### YAML Definitions

**StorageClass:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class 
provisioner: disk.csi.azure.com 
parameters:
  skuname: Standard_LRS  
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```
```bash
oc apply storeageclass.yml
```

**PVC Using StorageClass:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvcs  
spec:
  storageClassName: my-storage-class  
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
```bash
oc apply pvcs.yml
```

#### Key Learnings

- StorageClasses allow dynamic storage provisioning.

---

### 5. Deploy a StatefulSet with Persistent Storage

#### Description

Deployed a StatefulSet to ensure pods maintain their identity and persistent storage.

#### YAML Definition

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
  namespace: labs
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - name: my-pvc
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: my-pvc
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```
```bash
oc apply statefulset.yml
```

#### Key Learnings

- StatefulSets maintain pod identities across restarts.

---

## Conclusion

This lab covered managing sensitive data with Secrets, storing configurations with ConfigMaps, provisioning persistent storage with PVs and PVCs, defining StorageClasses, and deploying StatefulSets for stateful applications.
---
<br>
<br>

# Lab 4: Reliability and Scaling

## Lab Objectives

- Implement health probes to ensure applications are running and ready.
- Set resource requests and limits for better resource management.
- Scale applications manually and automatically to handle traffic changes.

## Tasks Completed

### 1. Implement Liveness and Readiness Probes

**Description:**

- Configured probes to check if the application is alive and ready.
- Used `livenessProbe` to restart unresponsive containers.
- Used `readinessProbe` to stop traffic to unready pods.

**Key Learnings:**

- Probes help improve application reliability and availability.

**YAML File:** `liveness-readiness.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reliability-deployment
spec:
  replicas: 3
  selector:  
    matchLabels:
      app: reliability-app
  template:
    metadata:
      labels:
        app: reliability-app
    spec:
      containers:
      - name: reliability-container
        image: nginx
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```
```bash
oc apply -f liveness-readiness.yaml
```

---

### 2. Set Resource Requests and Limits

**Description:**

- Set CPU and memory requests to ensure sufficient resources for the app.
- Defined limits to prevent resource overuse.

**Key Learnings:**

- Prevents resource starvation and improves cluster efficiency.

**YAML File:** `resource-limits.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: resource-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
```bash
oc apply -f resource-limits.yaml
```

---

### 3. Scale Deployment and Set Autoscaling

**Description:**

- Scaled deployment manually to 5 replicas.
- Configured autoscaling to adjust replicas based on CPU usage.

**Key Learnings:**

- Autoscaling ensures high availability during traffic spikes.

**Manual Scaling Command:**

```bash
oc scale deployment reliability-deployment --replicas=5
```

**YAML File:** `autoscaler.yaml`

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: reliability-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: reliability-deployment
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
```bash
oc apply -f autoscaler.yaml
```
---

## Conclusion for Lab 4

This lab improved my understanding of making applications reliable and scalable by using health probes, setting resource limits, and configuring autoscaling.
---
<br>
<br>

# Lab 5: Application Updates and Rollbacks in OpenShift

## Lab Objectives
- Update applications without downtime.
- Roll back failed deployments quickly.
- Use ImageStreams to automate deployments when new images are available.

## Tasks Completed

### 1. Update an Application and Monitor the Rollout Status

#### Description:
Updated an application with a new container image and monitored the rollout to ensure success.

#### Command Used:
```bash
oc set image deployment/my-app my-container=nginx:1.26.2
```

#### Checked Deployment Status:
```bash
oc rollout status deployment/my-app
```

#### Key Learnings:
- How to update a running application.

---

### 2. Simulate a Failed Deployment and Perform a Rollback

#### Description:
Simulated a failed deployment using an invalid image and then rolled back to the last successful version.

#### Commands Used:
```bash
oc set image deployment/my-app my-container=invalid-image
oc rollout status deployment/my-app
oc rollout undo deployment/my-app
```

#### Key Learnings:
- OpenShift pauses failed rollouts automatically.
- `oc rollout undo` quickly restores a working version.

---

### 3. Automate Deployments Using ImageStreams and Triggers

#### Description:
Created an ImageStream and set up automatic updates when a new image is available.

#### Commands Used:
```bash
oc create imagestream my-app-stream
oc tag nginx:1.26.3 my-app-stream
```

#### YAML File Used (`imagestream.yaml`):
```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: my-app-stream
spec:
  tags:
    - name: 1.26.3
      from:
        kind: ImageStreamTag
        name: nginx:1.26.3
      importPolicy:
        automatic: true
```

#### Apply the YAML File:
```bash
oc apply -f imagestream.yaml
```

#### Key Learnings:
- ImageStreams manage and track container images.
- Automatic triggers ensure deployments stay up to date.

---

## Conclusion for Lab 5
This lab improved my understanding of application updates, rollbacks, and automation in OpenShift.










