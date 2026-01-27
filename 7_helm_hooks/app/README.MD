# Notes
## Hooks
Chart hooks allow execution of custom logic at certain points in a release lifecycle and are K8s resources with special annotations:
```
pre-/post-install: Executes after templates are rendered but before any resources are created / Executes after all resources are loaded into Kubernetes
pre-/post-delete: Executes on a deletion request before any resources are deleted from Kubernetes / Executes on a deletion request after all release resources have been deleted.
pre-/post-upgrade: Executes on an upgrade request after templates are rendered, BUT BEFORE any resources are updated / Executes on an upgrade request after all resources have been upgraded.
pre-/post-rollback: Executes on a rollback request after templates are rendered but before any resources are rolled back / Executes on a rollback request after all resources have been modified. 
test: Executes when helm test subcommand is invoked.
```

A typical hook is a Kubernetes yaml file with annotations.
```
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}"
  labels:
    ...
  annotations:
    "helm.sh/hook": post-install, post-upgrade
    "helm.sh/hook-weight": "-1" #defaults to 0
    "helm.sh/hook-delete-policy": hook-succeeded  #defaults to "before-hook-creation"
```


As a best practice, specify your hooks in a hooks/ folder and whenever you run:
```
helm upgrade <app-name> <folder or ".">
```
The hooks run