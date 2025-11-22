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

```shell
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.12.2.15
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-2x3-api-svc.earth:4546
```

### Pods

```shell
k run resource-checker --image=httpd:alpine -n=limit --dry-run=client -o=yaml > resource-checker.yaml
vim resource-checker.yaml
ka resource-checker.yaml
```

### Deployments

```shell
k -n sun create deploy -h
k -n sun create deploy sunny --image=nginx:1.17.3-alpine --dry-run=client -o=yaml > sunny.yaml
vim sunny.yaml
```

### Services

```shell
k -n sun expose -h
k -n sun expose deploy sunny --name sun-srv --port 9999 --target-port 80
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
