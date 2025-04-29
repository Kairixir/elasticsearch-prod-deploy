# Minikube setup for ECK

## Start minikube and [enable storage provisioner][2]

/_I believe the docs are incorrect. Last update is 2022, and the Rancher
provisioner seems to be provisioner for the service rancher, not for local
provisioning. I believe it due to output of `minikube addons list`_/

```bash
| storage-provisioner         | minikube | enabled ✅   | minikube             |
| storage-provisioner-gluster | minikube | disabled     | 3rd party (Gluster)  |
| storage-provisioner-rancher | minikube | disabled     | 3rd party (Rancher)  |
```

```bash
minikube start --nodes 3 --cpus 2 --memory 8G
# From docs but I believe incorrect
# minikube addons enable storage-provisioner-rancher
minikube addons enable storage-povisioner
minikube addons enable default-storageclass
```

[2]: https://minikube.sigs.k8s.io/docs/tutorials/local_path_provisioner/

## Deploy local path provisioner

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.31/deploy/local-path-storage.yaml
```

## [Install ECK operator](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/install-using-yaml-manifest-quickstart)

```bash
# Install custom resource definitions
kubectl apply -f https://download.elastic.co/downloads/eck/3.0.0/crds.yaml

# Install the operator
kubectl apply -f https://download.elastic.co/downloads/eck/3.0.0/operator.yaml

# Monitor operator's setup in logs
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator

```
