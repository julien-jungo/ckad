# CKAD Cheat Sheet

## Docs

<https://kubernetes.io/docs/home/>

## Copy/Paste

### Browser

```shell
CTRL+C
CTRL+V
```

### Terminal

```shell
CTRL+SHIFT+C
CTRL+SHIFT+V
```

## Vim Settings

```text
# ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
```

## SSH

```shell
ssh node01
```

## Aliases

```shell
alias k='kubectl'
alias ka='kubectl apply -f'
```

## Commands

### Contexts

```shell
k config -h
k config get-contexts
k config set-context --current --namespace=development
k config set-context developer --cluster=kubernetes --namespace=development --user=martin
k config use-context developer
```

### Checks

#### Network

```shell
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.12.2.15
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-2x3-api-svc.earth:4546
```

#### Pods

```shell
k exec -it pod1 -- /bin/sh
```

### Create

#### Pods

```shell
k run resource-checker --image=httpd:alpine -n=limit --dry-run=client -o=yaml > resource-checker.yaml
vim resource-checker.yaml
ka resource-checker.yaml
```

#### Deployments

```shell
k -n sun create deploy -h
k -n sun create deploy sunny --image=nginx:1.17.3-alpine --dry-run=client -o=yaml > sunny.yaml
vim sunny.yaml
```

#### Services

```shell
k -n sun expose -h
k -n sun expose deploy sunny --name sun-srv --port 9999 --target-port 80
```

### Rollouts

#### Rolling Update

```shell
k edit deploy wonderful-v1
```

#### Blue-Green

```shell
k get deploy wonderful-v1 -o yaml > wonderful-v2.yaml
vim wonderful-v2.yaml
k create -f wonderful-v2.yaml
k edit svc wonderful
k scale deploy wonderful-v1 --replicas 0
```

#### Canary

```shell
k get deploy wonderful-v1 -o yaml > wonderful-v2.yaml
vim wonderful-v2.yaml
k create -f wonderful-v2.yaml
k scale deploy wonderful-v2 --replicas 2
k scale deploy wonderful-v1 --replicas 8
```

### Helm

```shell
helm list -A
helm uninstall apiserver -n team-yellow
helm install dev falcosecurity/falco -n team-yellow
```

## Objects

### Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  namespace: development
  labels:
    run: jekyll
spec:
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
  initContainers:
    - name: copy-jekyll-site
      image: gcr.io/kodekloud/customimage/jekyll
      command: ["rm", "-rf", "/site/*", "&&", "jekyll", "new", "/site", "&&", "cd", "/site", "&&", "bundle", "install"]
      volumeMounts:
        - mountPath: "/site"
          name: site
  containers:
    - name: jekyll
      image: "gcr.io/kodekloud/customimage/jekyll-serve"
      command: ["cd", "/site", "&&", "bundle", "install", "&&", "bundle", "exec", "jekyll", "serve", "--host", "0.0.0.0", "--port", "4000"]
      ports:
        - containerPort: 4000
      volumeMounts:
        - mountPath: "/site"
          name: site
```

#### Resources

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: resource-checker
  name: resource-checker
  namespace: limit
spec:
  containers:
  - image: httpd:alpine
    name: my-container
    resources:
      limits:
        cpu: 300m
        memory: 30Mi
      requests:
        cpu: 30m
        memory: 30Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### Readiness Probe

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: space-alien-welcome-message-generator
  name: space-alien-welcome-message-generator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: space-alien-welcome-message-generator
  strategy: {}
  template:
    metadata:
      labels:
        app: space-alien-welcome-message-generator
    spec:
      containers:
      - image: httpd:alpine
        name: httpd
        resources: {}
        readinessProbe:
          exec:
            command:
            - stat
            - /tmp/ready
          initialDelaySeconds: 10
          periodSeconds: 5
status: {}
```

### Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jekyll-node-service
  namespace: development
spec:
  type: NodePort
  selector:
    run: jekyll
  ports:
    - port: 4000
      targetPort: 4000
      nodePort: 30097
```

### Config Maps

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: trauerweide
data:
  tree: trauerweide
```

#### Mounting

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: nginx:alpine
    name: pod1
    resources: {}
    env:
    - name: TREE1
      valueFrom:
        configMapKeyRef:
          name: trauerweide
          key: tree
    volumeMounts:
    - name: birke
      mountPath: "/etc/birke"
  volumes:
  - name: birke
    configMap:
      name: birke
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Persistent Volumes

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jekyll-site
spec:
  storageClassName: "local-storage"
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 1Gi
  hostPath:
    path: "/mnt/data"
```

### Persistent Volume Claims

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

### Roles

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: development
rules:
  - apiGroups: [""]
    resources: ["services", "persistentvolumeclaims", "pods"]
    verbs: ["*"]
```

### Role Bindings

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-rolebinding
  namespace: development
subjects:
  - kind: User
    name: martin
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # k get ingressclass
  rules:
  - host: world.universe.mine
    http:
      paths:
      - backend:
          service:
            name: europe
            port:
              number: 80
        path: /europe
        pathType: Prefix
      - backend:
          service:
            name: asia
            port:
              number: 80
        path: /asia
        pathType: Prefix
status:
  loadBalancer: {}
```

### Custom Resource Definitions (CRD)

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: shopping-items.beta.killercoda.com
spec:
  group: beta.killercoda.com
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
                description:
                  type: string
                dueDate:
                  type: string
  scope: Namespaced
  names:
    plural: shopping-items
    singular: shopping-item
    kind: ShoppingItem
```

#### Custom Objects

```yaml
apiVersion: beta.killercoda.com/v1
kind: ShoppingItem
metadata:
  name: bananas
spec:
  dueDate: tomorrow
  description: buy yellow ones
```

## ~/.kube/config

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ...
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: development
    user: martin
  name: developer
- context:
    cluster: kubernetes
    namespace: development
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: martin
  user:
    client-key: /root/martin.key
    client-certificate: /root/martin.crt
- name: kubernetes-admin
  user:
    client-certificate-data: ...
    client-key-data: ...
```
