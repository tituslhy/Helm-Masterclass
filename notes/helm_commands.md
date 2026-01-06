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
helm install <helm chart of interest>
```

## To check kubectl version
```
kubectl version
```

## To check what kubernetes is running on
```
kubectl config current-context
```

## To start pod
```
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

## To get metadata about a specific pod
Run `kubectl get pod` first to see running pods. Then run:

```
kubectl describe pod local-wp-wordpress-db9c644bb-8h4sz 
```

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