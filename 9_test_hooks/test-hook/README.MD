# This repo is only concerned with the tests folder
Tests are invoked by running the following command:
```
helm test <release_name>
```

Tests are run against a release. So you must first run:
```
helm install <release name <helm chart folder>
```

OR

```
helm upgrade <release> <folder>
```

By default pods are left dangling even if the job is completed:
```
➜  Helm-Masterclass git:(main) ✗ kubectl get pod        
NAME                               READY   STATUS      RESTARTS   AGE
local-test-hook-68cd94cdd5-d6tfq   1/1     Running     0          98s
local-test-hook-test-connection    0/1     Completed   0          32s
```

You can also run:
```
kubectl get logs <pod_name>
```
to also see why tests fail

### Example

```
➜  test-hook git:(main) ✗ helm test local-test-hook     
NAME: local-test-hook
LAST DEPLOYED: Wed Jan 28 22:04:36 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE:     local-test-hook-test-connection
Last Started:   Wed Jan 28 22:05:42 2026
Last Completed: Wed Jan 28 22:05:51 2026
Phase:          Succeeded
```

## On the default test-connection.yaml
It isn't standard practice to just place every test inside a test-connection.yaml file. Rather, it's more common to create multiple shell scripts placed inside a test-scripts folder, then create a config-map yaml file to include it, and then run it in the test-connection.yaml file.

### Note 
yaml, yml, tpl or txt files are only allowed in the templates folder. So your .sh files must be created outside the templates folder.