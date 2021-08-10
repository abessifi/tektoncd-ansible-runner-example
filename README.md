# Tektoncd Ansible Runner Examples on OpenShift

A repo to hold Ansible runner examples for the Tektoncd Task `ansible-runner`


## Common Tasks

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

## Ansible Runner

Create the ansible-runner Task:

```shell
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/ansible-runner/0.1/ansible-runner.yaml
```

Verify the created tasks:

```shell
tkn task ls
```

## Examples

Do the git clone of the examples repository:

```shell
tkn clustertask start git-clone \
  --workspace=name=output,claimName=ansible-playbooks \
  --param=url=https://github.com/abessifi/tektoncd-ansible-runner-example \
  --param=revision=master \
  --param=deleteExisting=true \
  --showlog
```

or (if the git-clone is not predefined as a clustertask task):

```shell
tkn task start git-clone \
  --workspace=name=output,claimName=ansible-playbooks \
  --param=url=https://github.com/abessifi/tektoncd-ansible-runner-example \
  --param=revision=master \
  --param=deleteExisting=true \
  --showlog
```

Check the corresponding `taskrun`:

```shell
tkn taskrun ls
```

### Service Account

As we will do get, list and create on the namespace, lets use a service account that has right RBAC:

```shell
oc apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/ansible-runner/0.1/support/ansible-deployer.yaml

oc get sa
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
