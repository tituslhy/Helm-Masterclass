## List repo
```
helm repo list
```

## Search repo
```
helm search repo <keyword>
```

For example: `helm search repo wordpress`

## Search across charts
```
helm search chart <keyword>
```
For example: `helm search chart nginx`


## Install helm chart
```
helm install <release-name> <helm-repo>/<helm chart> --version=<version-of-interest>
```

The `--version` flag is optional. The default will be the latest version.

## To check kubectl version
```
kubectl version
```

## To check what kubernetes is running on
```
kubectl config current-context
```

## To get pod
```
kubectl get pod
```

Output:
```
NAME                                 READY   STATUS    RESTARTS      AGE
local-wp-mariadb-0                   1/1     Running   0             2m40s
local-wp-wordpress-db9c644bb-8h4sz   1/1     Running   1 (75s ago)   2m40s
```

## To get services
```
kubectl get svc
```

Output:
```
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes                  ClusterIP      10.96.0.1        <none>        443/TCP                      182d
local-wp-mariadb            ClusterIP      10.102.203.206   <none>        3306/TCP                     2m57s
local-wp-mariadb-headless   ClusterIP      None             <none>        3306/TCP                     2m57s
local-wp-wordpress          LoadBalancer   10.96.148.123    <pending>     80:32277/TCP,443:32340/TCP   2m57s
```

## To get secrets
```
kubectl get secret
```

Output:
```
tituslim@tituslhyM3M Helm-Masterclass % kubectl get secret                        
NAME                             TYPE                 DATA   AGE
local-wp-mariadb                 Opaque               2      3m41s
local-wp-wordpress               Opaque               1      3m41s
sh.helm.release.v1.local-wp.v1   helm.sh/release.v1   1      3m41s
```

## To get a specific secret
```
kubectl get secret <secret_name> -o jsonpath'{<path of password>}' | base64 -d
```
This gets a secret's actual value according to the json path (initially returned as a base64 string) which we must decode.

For example:
```
kubectl get secret local-wp-wordpress -o jsonpath='{.data.wordpress-password}' | base64 -d
```
Where `.data` refers to the root level, and `.wordpress-password` is the key of interest.

##### Note
- The password will be returned in the terminal and includes all the characters except the '%' sign at the end.
- It's very important to look at the default settings of the artifact on Artifact Hub. It'll provide information on defaults

## To describe a secret
```
kubectl describe secret <secret_name>
```

## To get metadata about a specific pod
Run `kubectl get pod` first to see running pods. Then run:

```
kubectl describe pod <pod-name>
```

Alternatively, run:
```
kubectl get pod --watch
```
To get live updates to see when your pods are live.

## To get deployed pods
```
kubectl get deploy
```

Output:
```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
local-wp-wordpress   1/1     1            1           4m28s
```

## To expose deployment
This exposes deployment for external connection
```
kubectl expose deploy local-wp-wordpress --type=NodePort --name=local-wp
```

The `--name` flag is used to deconflict names if the desired name 'local-wp-wordpress' is already in use.

## To connect deployment to minikube
```
minikube service <app-name>
```

So in this case: `minikube service local-wp`

## To retrieve the values in the Chart.yaml folder
```
helm show values <repo>/<chart>
```

OR

```
helm get values <release-name> --all
```

For example:
```
helm show values bitnami/wordpress
```

## To get user supplied values to the helm chart
```
helm get values <name-of-release>
```

## To get notes of the release
```
helm get notes <name-of-release>
```

## Get metadata of release
```
helm get metadata <name-of-release>
```

## Get a list of releases
```
helm list
```

## To uninstall a release
```
helm uninstall <release-name>
```

This uninstalls the release and removes the secrets so if you run `kubectl get pod` you should not see the release name there.

But you will still see the service if you run `kubectl get svc`. That's because we created this service to expose the helm release. So we have to delete this with:

```
kubectl delete svc <release-name>
```

### You also need to check for persistent volumes or persistent volume claims
Some applications have persistent volumes! The generated secrets might have been deleted in kubectl but are persisted in the persistent volume claim. 

The previously generated secrets therefore still remain as the correct passwords - so when you reinstall a helm chart, the newly generated passwords won't work, and your application will fail to start.

Run:
```
kubectl get pv,pvc
```

To delete this:
```
kubectl delete pvc <name-of-persistent-volume-claim> 
```

```
kubectl delete pv <name-of-persistent-volume>
```

### To double check before deleting
```
kubectl describe pvc <name of persistent volume claim>
```

## To view logs of a pod
```
kubectl logs <pod-of-interest>
```

## To set custom values (passwords) via the Helm CLI
This is not recommended if there are many values to set. It'll be more practical to download the artifacts from Artifact Hub and editing the yaml files.

```
helm install <release-name> <repo>/<chart> --version=.... \
--set "<key_name>=<value_name>"
```

For example:
```
helm install local-wp bitnami/wordpress --version=23.1.20 \
    --set "mariadb.auth.rootPassword=myawesomepassword" \
    --set "mariadb.auth.password=myuserpassword"
```

The better way:
```
EXPORT mariadbRootPassword=...
EXPORT mariadbUserPassword=...

helm install local-wp bitnami/wordpress --version=23.1.20 \
    --set "mariadb.auth.rootPassword=$mariadbRootPassword" \
    --set "mariadb.auth.password=$mariadbUserPassword"
```

The even better way:
1. Create a custom.yaml file with all the key-value pairs
2. But don't add the literal password into the file - do not commit the secret!
3. Use a placeholder value:
```yaml
existingSecret: custom-wp-credentials
```
Where `custom-wp-credentials` is a variable name
4. Then create the variable in kubectl using:
```
kubectl create secret generic custom-wp-credentials --from-literal <key_of_interest>=<value_of_interest>
```

For example:
```
kubectl create secret generic custom-wp-credentials --from-literal wordpress-password=lauropassword
```

Then running
```
kubectl get secret
```
will show that the secret has been created.

and running:
```
kubectl get secret custom-wp-credentials -o jsonpath='{.data.wordpress-password}' | base64 -d
```
will return the exact password.

Then when starting your app,
```
helm install <release-name> <repo>/<chart> --version=<version_number> -f path/to/file.yaml
```

Double check that your secrets have passed through:
```
helm get values <release name>
```

## Upgrade helm releases
After updating the values in a yaml file, run:
```
helm upgrade --reuse-values --values path/to/file.yaml <release-name> <repo-name>/<chart-name> --version <version-number> --atomic --cleanup-on-fail --debug --timeout 2m
```

The `--reuse-values` flag ensures that we use the same values that have already been deployed in the previous release for the flags that have not been edited.

The `--atomic` flag rollsback to the previous deployment if the target deployment has an issue.

The `cleanup-on-fail` removes resources that were spawned by the failed deployment.

The `debug` streams logs.

The `--timeout` flag sets the timeout duration. The default is 5 minutes so reducing the timeout will allow us to terminate the deployment faster on failure.

## To see release history
```
helm history <release-name>
```

## Helm rollback
To rollback an update:
```
helm rollback <release-name> <revision_number>
```
Check replica sets whenever rolling back, because resources were created even for failed deployments. You have to manually clean up these resources. Replica sets are not directly managed by helm but by the deployment - i.e. once the deployment runs (even if it fails) these replica sets are created.

To view replica sets:
```
kubectl get rs
```

output:
```
NAME                            DESIRED   CURRENT   READY   AGE
local-wp-wordpress-5c975958d7   2         2         2       66m
local-wp-wordpress-686b4cd69    0         0         0       2m35s
```

Notice that there is one replica set that has zeros for everywhere. This is the failed helm upgrade. You can verify that there is only one deployment:
```
kubectl get deploy
```

Remember to delete unused replica sets:
```
kubectl delete rs <replica-set>
```