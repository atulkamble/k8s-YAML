# Kubernetes YAML Templates – Complete Practice Guide

This guide combines Kubernetes resources, their significance, why we use them, and practical YAML code for Minikube, kubeadm, AWS EKS, Azure AKS, and GKE.

Each template includes a short explanation and a usable example. Some advanced resources require additional controllers or add-ons.

## 1. Pod – `pod.yaml`

Significance: Smallest deployable unit in Kubernetes.

Why use it: To run one or more containers sharing networking and storage.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
```

## 2. Multi-container Pod – `multi-container-pod.yaml`

Significance: Runs multiple containers inside one Pod.

Why use it: For tightly coupled application and helper containers.

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
    - name: helper
      image: busybox:1.36
      command: ["sh", "-c", "while true; do date; sleep 30; done"]
```

## 3. Namespace – `namespace.yaml`

Significance: Provides logical isolation within a cluster.

Why use it: To separate development, testing, and production workloads.

```
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

## 4. ReplicaSet – `replicaset.yaml`

Significance: Maintains the required number of Pod replicas.

Why use it: To replace failed Pods automatically. Normally managed through a Deployment.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

## 5. Deployment – `deployment.yaml`

Significance: Manages application lifecycle, scaling, and updates.

Why use it: For stateless applications requiring high availability and rolling updates.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 500m
              memory: 128Mi
```

## 6. ClusterIP – `clusterip.yaml`

Significance: Exposes an application internally.

Why use it: For internal communication between services and Pods.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

## 7. NodePort – `nodeport.yaml`

Significance: Exposes a Service on a port of each node.

Why use it: For simple external access during practice and testing.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Access using `http://NODE-IP:30080` when networking and firewall rules permit.

## 8. LoadBalancer – `loadbalancer.yaml`

Significance: Exposes applications through a load balancer.

Why use it: To provide external or internal access to applications on cloud platforms.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

Requires a supported load-balancer implementation.

## 9. ConfigMap – `configmap.yaml`

Significance: Stores non-sensitive configuration separately from application images.

Why use it: To configure applications without rebuilding images.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
```

## 10. Secret – `secret.yaml`

Significance: Stores sensitive application configuration.

Why use it: For database passwords, credentials, and API keys.

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USER: "webapp"
  DB_PASSWORD: "replace-with-secure-password"
```

For production, use a secrets manager or external secrets controller. Kubernetes Secrets are not automatically encrypted in etcd unless encryption at rest is configured.

## 11. PersistentVolume – `pv.yaml`

Significance: Represents persistent storage in a cluster.

Why use it: To preserve application data independently of Pod restarts.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
```

This `hostPath` example is for single-node labs only.

## 12. PersistentVolumeClaim – `pvc.yaml`

Significance: Requests persistent storage.

Why use it: To attach storage to applications without directly managing physical disks.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 5Gi
```

## 13. StorageClass – `storageclass.yaml`

Significance: Defines how Kubernetes provisions storage.

Why use it: For dynamic volume creation through a storage driver.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-storage
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
parameters:
  type: gp3
```

AWS EKS example: requires a compatible EBS CSI driver and appropriate permissions. Use a different provisioner for AKS, GKE, or other environments.

## 14. StatefulSet – `statefulset.yaml`

Significance: Maintains stable Pod identities and supports persistent storage.

Why use it: For databases and other stateful applications.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
    - port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  serviceName: nginx-headless
  replicas: 2
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

## 15. DaemonSet – `daemonset.yaml`

Significance: Runs one Pod on each eligible node.

Why use it: For log collectors, monitoring agents, and node-level utilities.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
spec:
  selector:
    matchLabels:
      app: nginx-daemon
  template:
    metadata:
      labels:
        app: nginx-daemon
    spec:
      containers:
        - name: nginx
          image: nginx:stable
```

## 16. Job – `job.yaml`

Significance: Executes tasks until successful completion.

Why use it: For database migrations, scripts, and batch processing.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: hello
          image: busybox:1.36
          command:
            - sh
            - -c
            - echo "Hello Kubernetes"
```

## 17. CronJob – `cronjob.yaml`

Significance: Runs tasks on a schedule.

Why use it: For backups, periodic reports, and cleanup jobs.

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: busybox:1.36
              command:
                - sh
                - -c
                - echo "Scheduled backup task"
```

This demonstration runs every five minutes; it does not perform a real backup.

## 18. Horizontal Pod Autoscaler – `hpa.yaml`

Significance: Automatically adjusts the number of Pods.

Why use it: To handle changing application workloads.

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Requires CPU requests and a working resource metrics pipeline.

## 19. Vertical Pod Autoscaler – `vpa.yaml`

Significance: Recommends or adjusts CPU and memory resources.

Why use it: To right-size containers and reduce resource waste.

```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Off"
```

Requires VPA installation. `Off` provides recommendations without automatically modifying Pods.

## 20. Ingress – `ingress.yaml`

Significance: Manages HTTP/HTTPS routing into the cluster.

Why use it: To expose multiple applications using hostnames and URL paths.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: nginx.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-clusterip
                port:
                  number: 80
```

Requires an installed Ingress controller and appropriate DNS configuration.

## 21. NetworkPolicy – `networkpolicy.yaml`

Significance: Controls network traffic between Pods.

Why use it: To restrict unauthorized application communication.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-network-policy
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
```

Requires a CNI plugin that supports NetworkPolicy enforcement.

## 22. ServiceAccount – `serviceaccount.yaml`

Significance: Provides workload identity inside Kubernetes.

Why use it: To allow applications to authenticate to the Kubernetes API.

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-serviceaccount
```

## 23. Role – `role.yaml`

Significance: Defines namespace-level permissions.

Why use it: To grant controlled access to Kubernetes resources.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

## 24. RoleBinding – `rolebinding.yaml`

Significance: Assigns a Role to an identity.

Why use it: To grant users or applications specific permissions within a namespace.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
subjects:
  - kind: ServiceAccount
    name: app-serviceaccount
    namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## 25. ClusterRole – `clusterrole.yaml`

Significance: Defines permissions across namespaces or for cluster-scoped resources.

Why use it: To manage access to resources such as nodes.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
```

## 26. ClusterRoleBinding – `clusterrolebinding.yaml`

Significance: Grants ClusterRole permissions across the cluster.

Why use it: To authorize cluster-wide administrative or monitoring tasks.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
  - kind: ServiceAccount
    name: app-serviceaccount
    namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

## 27. ResourceQuota – `resourcequota.yaml`

Significance: Limits total resource consumption in a namespace.

Why use it: To prevent teams or applications from consuming excessive resources.

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    pods: "10"
```

## 28. LimitRange – `limitrange.yaml`

Significance: Defines default and allowed resource limits.

Why use it: To control CPU and memory usage for individual containers.

```
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
spec:
  limits:
    - type: Container
      defaultRequest:
        cpu: 100m
        memory: 64Mi
      default:
        cpu: 500m
        memory: 256Mi
```

## 29. PodDisruptionBudget – `pdb.yaml`

Significance: Controls voluntary Pod disruptions.

Why use it: To maintain application availability during node drains and maintenance.

```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: nginx
```

## 30. PriorityClass – `priorityclass.yaml`

Significance: Defines Pod scheduling priority.

Why use it: To prioritize important workloads during resource shortages.

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "High priority application workloads"
```

## 31. Gateway API – `gateway.yaml`

Significance: Provides a flexible networking API for application traffic.

Why use it: To manage traffic entry points and advanced routing.

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
spec:
  gatewayClassName: example-gateway-class
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

Replace `example-gateway-class` with a GatewayClass provided by your installed Gateway controller.

## 32. HTTPRoute – `httproute.yaml`

Significance: Defines HTTP traffic routing rules.

Why use it: To route requests to Kubernetes Services using Gateway API.

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: nginx-route
spec:
  parentRefs:
    - name: web-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: nginx-clusterip
          port: 80
```

Gateway API CRDs and a compatible controller must be installed.

# Additional Kubernetes configuration templates

These are generally fields within Pod or workload specifications, rather than standalone Kubernetes resources.

## 33–35. Health probes – `probes.yaml`

Significance: Detects application health and readiness.

Why use it: To restart unhealthy containers and prevent traffic from reaching unready applications.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-healthcheck
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80

      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10

      readinessProbe:
        httpGet:
          path: /
          port: 80
        periodSeconds: 5

      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30
        periodSeconds: 5
```

## 36. Init Container – `init-container.yaml`

Significance: Runs initialization tasks before the main application.

Why use it: To prepare configuration, dependencies, or application data.

```
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
    - name: init
      image: busybox:1.36
      command: ["sh", "-c", "echo Initializing"]
  containers:
    - name: nginx
      image: nginx:stable
```

## 37. Sidecar Container – `sidecar.yaml`

Significance: Provides supporting functionality alongside the application.

Why use it: For logging, proxying, and other helper services.

```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
spec:
  containers:
    - name: nginx
      image: nginx:stable
    - name: helper
      image: busybox:1.36
      command:
        - sh
        - -c
        - while true; do echo "Helper running"; sleep 30; done
```

This demonstrates a regular helper container. Kubernetes also supports native sidecars through restartable init containers.

## 38. NodeSelector – `nodeselector.yaml`

Significance: Schedules Pods on nodes with matching labels.

Why use it: To run workloads on specific node groups.

```
apiVersion: v1
kind: Pod
metadata:
  name: node-selector-demo
spec:
  nodeSelector:
    workload: apps
  containers:
    - name: nginx
      image: nginx:stable
```

Requires a node labeled `workload=apps`.

## 39. Node Affinity – `node-affinity.yaml`

Significance: Provides flexible node scheduling rules.

Why use it: To select nodes based on hardware, zones, or custom labels.

```
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-demo
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: workload
                operator: In
                values:
                  - apps
  containers:
    - name: nginx
      image: nginx:stable
```

## 40. Pod Affinity – `pod-affinity.yaml`

Significance: Places Pods near other matching Pods.

Why use it: To colocate related workloads when appropriate.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-demo
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: backend
            topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:stable
```

## 41. Pod Anti-Affinity – `pod-anti-affinity.yaml`

Significance: Spreads Pods across different nodes.

Why use it: To reduce the risk of multiple replicas failing together.

```
apiVersion: v1
kind: Pod
metadata:
  name: anti-affinity-demo
  labels:
    app: web
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchLabels:
                app: web
            topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx:stable
```

## 42. Taints and Tolerations – `tolerations.yaml`

Significance: Controls which workloads are permitted on tainted nodes.

Why use it: To reserve nodes for specialized workloads.

```
apiVersion: v1
kind: Pod
metadata:
  name: toleration-demo
spec:
  tolerations:
    - key: dedicated
      operator: Equal
      value: apps
      effect: NoSchedule
  containers:
    - name: nginx
      image: nginx:stable
```

Example node command:

```
kubectl taint nodes NODE_NAME dedicated=apps:NoSchedule
```

## 43. Topology Spread Constraints – `topology-spread.yaml`

Significance: Distributes application replicas across failure domains.

Why use it: To improve high availability across nodes or zones.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spread-demo
  template:
    metadata:
      labels:
        app: spread-demo
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: spread-demo
      containers:
        - name: nginx
          image: nginx:stable
```

## 44. SecurityContext – `securitycontext.yaml`

Significance: Defines Pod and container security settings.

Why use it: To reduce privileges and strengthen workload security.

```
apiVersion: v1
kind: Pod
metadata:
  name: security-demo
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 101
  containers:
    - name: nginx
      image: nginxinc/nginx-unprivileged:stable
      ports:
        - containerPort: 8080
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
        seccompProfile:
          type: RuntimeDefault
```

## 45. ImagePullSecrets – `imagepullsecret.yaml`

Significance: Authenticates pulls from private container registries.

Why use it: To deploy private Docker images.

```
apiVersion: v1
kind: Pod
metadata:
  name: private-image-demo
spec:
  imagePullSecrets:
    - name: registry-secret
  containers:
    - name: app
      image: registry.example.com/myapp:v1
```

Create the referenced secret using a secure credential source:

```
kubectl create secret docker-registry registry-secret \
  --docker-server=registry.example.com \
  --docker-username=USERNAME \
  --docker-password=TOKEN
```

## 46. Helm Chart – `templates/deployment.yaml`

Significance: Packages reusable Kubernetes applications.

Why use it: To deploy configurable applications across multiple environments.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Example `values.yaml`:

```
replicaCount: 3

image:
  repository: nginx
  tag: stable
```

## 47. Kustomize – `kustomization.yaml`

Significance: Customizes existing Kubernetes manifests.

Why use it: To manage environment-specific configurations without duplicating YAML.

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - clusterip.yaml

namespace: production

commonLabels:
  managed-by: kustomize
```

Create the namespace first, and ensure label transformations do not conflict with immutable Deployment selectors.

```
kubectl apply -f namespace.yaml
kubectl apply -k .
```

## 48. CustomResourceDefinition – `crd.yaml`

Significance: Extends Kubernetes with custom resource types.

Why use it: To support operators and custom Kubernetes APIs.

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.example.com
spec:
  group: example.com
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                domain:
                  type: string
```

Example custom resource, after installing the CRD:

```
apiVersion: example.com/v1
kind: Website
metadata:
  name: demo-website
spec:
  domain: demo.example.com
```

A CRD adds an API type; it does not automatically create a controller to act on those objects.

# Kubernetes practice commands

```
# Verify cluster
kubectl cluster-info
kubectl get nodes
kubectl get namespaces

# Deploy applications
kubectl apply -f pod.yaml
kubectl apply -f deployment.yaml
kubectl apply -f clusterip.yaml

# Inspect resources
kubectl get all
kubectl get pods -o wide
kubectl get deployments
kubectl get svc
kubectl get ingress
kubectl get pvc
kubectl get hpa

# Troubleshooting
kubectl describe pod mypod
kubectl logs mypod
kubectl get events --sort-by=.metadata.creationTimestamp

# Scaling
kubectl scale deployment nginx-deployment --replicas=5

# Rollout management
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment

# Test internal service access
kubectl port-forward svc/nginx-clusterip 8080:80

# Validate YAML
kubectl apply --dry-run=client -f deployment.yaml
kubectl diff -f deployment.yaml

# Delete resources
kubectl delete -f pod.yaml
kubectl delete -f deployment.yaml
```

## Recommended repository structure

```
kubernetes-templates/
├── 01-basics/
│   ├── pod.yaml
│   ├── multi-container-pod.yaml
│   └── namespace.yaml
├── 02-workloads/
│   ├── replicaset.yaml
│   ├── deployment.yaml
│   ├── statefulset.yaml
│   └── daemonset.yaml
├── 03-networking/
│   ├── clusterip.yaml
│   ├── nodeport.yaml
│   ├── loadbalancer.yaml
│   ├── ingress.yaml
│   ├── gateway.yaml
│   └── httproute.yaml
├── 04-configuration/
│   ├── configmap.yaml
│   └── secret.yaml
├── 05-storage/
│   ├── pv.yaml
│   ├── pvc.yaml
│   └── storageclass.yaml
├── 06-autoscaling/
│   ├── hpa.yaml
│   └── vpa.yaml
├── 07-security/
│   ├── serviceaccount.yaml
│   ├── role.yaml
│   ├── rolebinding.yaml
│   ├── clusterrole.yaml
│   ├── clusterrolebinding.yaml
│   └── networkpolicy.yaml
├── 08-jobs/
│   ├── job.yaml
│   └── cronjob.yaml
├── 09-scheduling/
│   ├── node-affinity.yaml
│   ├── pod-affinity.yaml
│   ├── pod-anti-affinity.yaml
│   └── tolerations.yaml
├── 10-resource-management/
│   ├── resourcequota.yaml
│   ├── limitrange.yaml
│   └── pdb.yaml
├── 11-advanced/
│   ├── probes.yaml
│   ├── securitycontext.yaml
│   └── crd.yaml
└── README.md
```

Important: These are individual practice examples, not one application to deploy all at once. Several templates intentionally use the same labels or resource names, and some require prerequisites. Practice them separately in suitable namespaces.

Reference: [Official Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/?utm_source=chatgpt.com).
