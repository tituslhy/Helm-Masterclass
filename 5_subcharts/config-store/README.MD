# Running the helm chart

To add the postgresql dependency. Go to Chart.yaml and add
```
dependencies:
  - name: postgresql
    version: '18.2.3'
    repository: 'https://charts.bitnami.com/bitnami'
```

Then run:
```
helm dependency update
```

This does the following:
- Updates dependencies,
- Pulls the helm chart for PostGreSQL in tar file format, storing it under "charts", and
- Creates a `Chart.lock` file to ensure that we always use the exact same version of the dependency.

Check:
```
helm dependency list
```

The command:
```
helm dependency build
```
rebuilds the charts folder based on the Chart.lock file - in case you accidentally delete the tar file. Note that this command does not update the `Chart.lock` file so do not use this command if you change the dependency version.

### Scopes
Don't use `global` in your values.yaml - this is a specially scoped value that injects the same value across all subcharts. Do this only if you need to pass a global variable to all sub charts.

But be careful! If there are global variables in the subcharts (for e.g. the PostGreSQL subchart), you'll end up overwriting it which will cause some problems.

## A note on PostGreSQL Chart
The release of version 16.6.0 changed the default value for the usePasswordFiles to true in the values.yaml file. This might break the upcoming implementation where we rely on environment variables instead of password files.

Simply set the option usePasswordFiles: false in the postgresql section of our chart's values.yaml file:
```
# Other values in our values.yaml 
 
postgresql:
  auth:
    # Keep other values...
    usePasswordFiles: false
  # Keep other values...
```

### How this links to Bitnami Charts
In your values.yaml file, under the postgresql configuration key, add the instructions to point to the bitnamilegacy repository. It should look like the following:
```
postgresql:
  image:
    registry: docker.io
    repository: bitnamilegacy/postgresql
```

Make sure that the configuration is correct by confirming that the values.yaml file is correctly used (you can run `helm template <config-store path> | grep bitnamilegacy/postgresql`, for example). 

This should show the following image: `docker.io/bitnamilegacy/postgresql:17.2.0-debian-12-r0`. 

This will enable you to continue using the same chart versions from the lectures for the moment, and you should still be able to install the applications on your Kubernetes cluster.