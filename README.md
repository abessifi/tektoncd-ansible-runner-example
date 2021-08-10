# Tektoncd Ansible Runner Examples

A repo to hold Ansible runner examples for the Tektoncd Task `ansible-runner`


## Common Tasks

### Kubernetes

Create a Namespace:

```shell
kubectl create ns funstuff
```

Define the `git-clone` Task:

```shell
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/git-clone/0.1/git-clone.yaml
```

Create a PVC:

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ansible-playbooks
  namespace: funstuff
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

### OpenShift

Create a Project:

```shell
oc new-project funstuff
```

In OpenShift 4 the `git-clone` task maybe already predefined as a `clustertask`. Check if it already exists before submitting the above YAML:

```shell
tkn clustertasks ls
tkn clustertasks describe git-clone
```

If the `git-clone` task is not defined:

```shell
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/task/git-clone/0.1/git-clone.yaml
```

Create a PVC:

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ansible-playbooks
  namespace: funstuff
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

## Examples

Run the following Task to clone this repository 

```shell
tkn task start git-clone \
  --workspace=name=output,claimName=ansible-playbooks \
  --param=url=https://github.com/kameshsampath/tektoncd-ansible-runner-example \
  --param=revision=master \
  --param=deleteExisting=true \
  --showlog
```

### Service Account

You need proper RBAC in Kubernetes to allow it to perform the example tasks:

```shell
kubectl apply -f  https://raw.githubusercontent.com/tektoncd-ansible-runner-example/master/kubernetes/ansible-deployer.yaml
```

### Listing pods

```shell
 tkn task start ansible-runner \
   --serviceaccount ansible-deployer-account \
   --param=project-dir=kubernetes \
   --param=args='-p list-pods.yml' \
   --workspace=name=runner-dir,claimName=ansible-playbooks \
   --showlog
```

### Create Deployment

```shell
 tkn task start ansible-runner \
   --serviceaccount ansible-deployer-account \
   --param=project-dir=kubernetes \
   --param=args='-p create-deployment.yml' \
   --workspace=name=runner-dir,claimName=ansible-playbooks \
   --showlog
```

### Create Service

```shell
 tkn task start ansible-runner \
   --serviceaccount ansible-deployer-account \
   --param=project-dir=kubernetes \
   --param=args='-p create-service.yml' \
   --workspace=name=runner-dir,claimName=ansible-playbooks \
   --showlog
```
