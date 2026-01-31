# SFTP Lab
A basic SFTP server based in `proftpd` and deployed using:
- kind
- Helm charts
- Kustomize overlays
- Argocd

# Step-by-step
## 1. Cluster creation
```bash
kind create cluster --name sftp-lab
```

## 2. Argocd installation
First of all, we have to create a specific namespace to `argocd`:
```bash
kubectl create namespace argocd
```

And install `argocd` into this namespace:
```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait until finished installation and check the status:
```bash
kubectl -n argocd get pods
```

By default, the `argocd` is exposed into 443 port. We can check it using:
```bash
kubectl -n argocd get services
```

And access to `argocd` through port-forward:
```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

And access to the UI open web-browser:
```bash
https://localhost:8080
````
## 3. Define GitOps folder structure

```bash
.
├── README.md
├── charts
│   └── proftpd
│       ├── Chart.yaml
│       ├── templates
│       │   ├── configmap.yaml
│       │   ├── deployment.yaml
│       │   ├── pvc.yaml
│       │   └── service.yaml
│       └── values.yaml
└── overlays
    └── dev
        ├── kustomization.yaml
        └── values-dev.yaml
```

## 4. Testing Helm charts and overlays
We can check the Helm charts without use `kustomize` command:
```bash
helm lint charts/proftpd
```
And the rendered manifest:
```bash
helm template proftpd charts/proftpd -f overlays/dev/values-dev.yaml > rendered.yaml
```
To perform a dry-run in local:
```bash
 kubectl apply --dry-run=client -f rendered.yaml
 ```

 And against the cluster:
 ```bash
 kubectl apply --dry-run=server -f rendered.yaml
 ```

## 5. Configure Argo CD
### a) Perform a `port-forward` to Argo CD service
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### b) Login to Argo CD
```bash
argocd login localhost:8080
```

### c) Add Github repository
```bash
argocd repo add https://github.com/ocriado91/sftp-lab.git
```

### d) Check repository
```bash
argocd repo list
```

### e) Apply Argo CD application
```bash
kubectl apply -f argocd/applications/sftp-dev.yaml
```

### f) Check Argo CD application
```bash
argocd app list
argocd app get sftp-dev
```

## 6. MetalLB installation

### a) Install MetalLB into cluster
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```

### b) Apply MetalLB into cluster
We can define the IP range through MetalLB configuration defined into `resources/metallb/metallb-config.yaml`:
```bash
kubectl apply -f resources/metallb/metallb-config.yaml
```
