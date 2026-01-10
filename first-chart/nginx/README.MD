# Creating our first chart

Each Helm chart deployment must have:
- Chart.yaml
- templates/deployment.yaml
- templates/service.yaml

If you are pulling an image off Dockerhub, key in the exact image number in the `appVersion` key of your `Chart.yaml` file

Once you're done creating the yaml files, run the following command from the `first-chart` folder:
```
helm template nginx
```
This generates the final templates that will be applied when installing the chart. This generates kubernetes manifests for the `deployment.yaml` and `setvice.yaml` files respectively.

Can also run the following command to lint nginx for a final check:
```
helm lint nginx
```

Then install it:
```
helm install local-nginx nginx
```

To check that your service is up:
```
kubectl get pods
```

```
kubectl get svc
```

To uninstall:
```
helm uninstall local-nginx
```

There isn't any persistent volume claim or persistent volume because it's just nginx.