# CKAD Cheat Sheet

## Docs

<https://kubernetes.io/docs/home/>

## Exam Environment

### Copy/Paste

#### Browser

```shell
CTRL+C
CTRL+V
```

#### Terminal

```shell
CTRL+SHIFT+C
CTRL+SHIFT+V
```

### Vim Settings

```text
# ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
```

### SSH

```shell
ssh node01
```

## Aliases

```shell
alias k='kubectl'
alias ka='kubectl apply -f'
```

## Exports

```shell
export NS=development
k get pods -n $NS
```

## Docker

### Dockerfile

```dockerfile
FROM busybox

ENTRYPOINT ["sleep"]

CMD ["3600"]
```

### Build

```shell
docker build -t busybox-sleep:latest .
```

### Tag

```shell
docker tag busybox-sleep:latest localhost:5000/busybox-sleep:latest
```

### List

```shell
docker images
```

### Push

```shell
docker push localhost:5000/busybox-sleep:latest
```

## Kustomize

TODO

## Helm

TODO

## Commands

### Base64

```shell
echo 'test' | base64
echo 'dGVzdAo=' | base64 -d
```

### Docker

```shell
docker build -t registry.killer.sh:5000/sun-cipher:v1-docker .
docker push registry.killer.sh:5000/sun-cipher:v1-docker
docker run -d --name sun-cipher registry.killer.sh:5000/sun-cipher:v1-docker
docker logs sun-cipher
```

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
k run tmp --restart=Never --rm -i --image=nginx:alpine -n space1 -- curl -m 5 microservice1.space2.svc.cluster.local
```

#### Pods

```shell
k exec -it pod1 -- /bin/sh
```

```shell
k port-forward pod1 8080:80
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

#### Ingress

```shell
k create ing web-ingress -n $NS --rule="web.example.com/*=web:80" --class=nginx
k get node -o wide
echo '172.30.1.2 web.example.com' >> /etc/hosts
k get svc --all-namespaces
curl web.example.com:30000
```

#### Secrets

```shell
k create secret generic app-secret --from-literal=password=pass123 -n session283884
```

#### Service Accounts

```shell
k create sa build-bot -n session283884
```

#### Roles

```shell
k -n session283884 create role pod-reader --verb=get,list,watch --resource=pods
```

#### Role Bindings

```shell
k -n session283884 create rolebinding read-pods-binding --role=pod-reader --serviceaccount=session283884:build-bot
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

### Rollbacks

```shell
k rollout history deploy api-new-c32
k rollout undo deploy api-new-c32
```

### Helm

```shell
helm list -A
helm uninstall apiserver -n team-yellow
helm install dev falcosecurity/falco -n team-yellow
helm upgrade internal-issue-report-apiv2 killershell/nginx -n mercury
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

#### Environment

##### Config Maps

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: env-pod
  name: env-pod
spec:
  containers:
  - image: busybox
    name: env-pod
    resources: {}
    command:
    - env
    envFrom:
    - configMapRef:
        name: env-config
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

##### Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: db-pod
  name: db-pod
  namespace: session283884
spec:
  containers:
  - image: busybox
    name: db-pod
    resources: {}
    command:
    - env
    envFrom:
    - secretRef:
        name: db-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### Shared Volumes

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: shared-pod
  name: shared-pod
  namespace: volumes
spec:
  containers:
  - image: nginx
    name: writer
    command:
    - sh
    - -c
    - |
      while true; do
        date >> /data/out.log
        sleep 1
      done
    volumeMounts:
    - name: data
      mountPath: /data
  - image: nginx
    name: reader
    command:
    - sh
    - -c
    - tail -f /data/out.log
    volumeMounts:
    - name: data
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: data
    emptyDir: {}
status: {}
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

### Jobs

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: neb-new-job
  namespace: neptune
  labels:
    id: awesome-job
spec:
  completions: 3
  parallelism: 2
  template:
    spec:
      containers:
      - name: neb-new-job-container
        image: busybox:1.31.0
        command: ["/bin/sh"]
        args: ["-c", "sleep 2 && echo done"]
      restartPolicy: Never
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

#### File Content

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-web-moon-html
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Web Moon Webpage</title>
    </head>
    <body>
      This is some great content.
    </body>
    </html>
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

### Storage Classes

TODO

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

### Service Accounts

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot
  namespace: session283884
```

### Roles

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: session283884
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

### Role Bindings

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: session283884
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: build-bot
  namespace: session283884
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

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  - to:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space2
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

### Service Accounts

TODO

### Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret1
data:
  user: dGVzdAo=
  pass: cHdkCg==
```

#### Environment Variables

TODO

## Kube Config

```yaml
# ~/.kube/config
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

## Kube API Server

```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.30.1.2:6443
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.30.1.2
    - ...
    - --enable-admission-plugins=NodeRestriction,LimitRanger,Priority
    - ...
    - --secure-port=6443
    - ...
```

## API Versions

### Kubernetes

```shell
k version
```

### API Groups

```shell
k api-resources
```

```shell
k explain deploy
```
