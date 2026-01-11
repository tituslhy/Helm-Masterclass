# Nginx charts with interpolation

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

## Note:
Try to use the following format for naming your service in your .yaml files:
```
{{ .Release.Name }}-{{ .Chart.Name }}
```

This ensures that each chart will be unique and you won't run into any errors trying to install charts with similar names.

The entire service.yaml file is wrapped in an if-else statement meaning the appliation will only be deployed if the values.yaml is set to deploy. If the service.yaml file is not rendered, then installing the helm chart does not deploy the service. This means you will not see your application if you run:
```
kubectl get svc
```