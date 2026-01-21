# Learning Go Templates

## Template functions and pipelines
> Visit [Helm documentation on functions](https://helm.sh/docs/chart_template_guide/function_list)

```
list: {{ list 1 2 3 }}
```

Returns a string representation if you run `helm template .` Note that this is string coercion
Output:
```
➜  4_templating-deep-dive git:(main) ✗ helm template .
---
# Source: templating-deep-dive/templates/sandbox.yaml
list: [1 2 3]
```

```
list: 
{{ list 1 2 3 | toYaml }} 
```

Returns a yaml:
```
---
# Source: templating-deep-dive/templates/sandbox.yaml
list: 
- 1
- 2
- 3
```

To add an indent to the print:
```
list: {{ toYaml (list 1 2 3) | nindent 2 }} 
```

OR
```
list: {{ list 1 2 3 | toYaml | nindent 2 }} 
```

Output:
```
---
# Source: templating-deep-dive/templates/sandbox.yaml
list: 
  - 1
  - 2
  - 3
```

Dictionaries are defined as `dict $key $val`. So to create nested dict:
```
my-dict: {{ dict "primitive" "my string" "complex" (dict "key1" "value1") }}
```

Remember to add indentation when printing toYaml so it creates a new line
```
my-dict: {{ dict "primitive" "my string" "complex" (dict "key1" "value1") | toYaml | nindent 2}}
```

## _helpers.tpl
Variable names get repeated quite often in helm charts and this is not recommended. Instead it is better to define a list of variables in a .tpl file and use them in helm charts.

It's convention to use the name of the chart as the first variable defined
```
{{- define "<name of helm chart>.fullName" -}}
<other statements here>
{{- end -}}
```

All variable definitions must be in string.

Then in your yaml files, you can write
{{ include "<name of helm chart>.fullName . }}

Using a $ in the _helpers.tpl defines a variable that can be used in helm
```
{{- define "templating-deep-dive.fullname" -}}
{{- $defaultName := printf "%s-%s" .Release.Name .Chart.Name }}
{{- .Values.customName | default $defaultName | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```
This sets a value for `defaultName` and changes it to `.Values.customName` if that variable exists, otherwise sets it as a default

The variable `$defaultName` is only accessible in the block of code above (within the {{define ... }} {{ end }} block).

You use `:=` for initial assignment.

## Add validation functions
```
{{- if and .Values.securityContext .Values.securityContext.enabled -}}
{{- $_ := required "securityContext.runAsUser is required when setting securityContext and enabled is true" .Values.securityContext.runAsUser -}}
{{- $_ := required "securityContext.fsGroup is required when setting securityContext and enabled is true" .Values.securityContext.fsGroup -}}
{{- end -}}
```
Use the special `required` function. Set `$_:=` to prevents the properties from being printed on the yaml. You could add this on top of the file (but if you do this you have to set a $_ variable) or in the yaml file directly, or in a different yaml file to be imported. It's not possible to set this in the _helper.tpl file because it won't be rendered properly.

If doing it inline, change the yaml file's values to:
```
runAsUser: {{ required "securityContext.runAsUser is required when setting securityContext and enabled is true" .runAsUser }}
fsGroup: {{ required "securityContext.fsGroup is required when setting securityContext and enabled is true" .fsGroup }}
```

If doing it in a different file, create a validation.yaml and paste:
```
{{- if and .Values.securityContext .Values.securityContext.enabled -}}
{{- $_ := required "securityContext.runAsUser is required when setting securityContext and enabled is true" .Values.securityContext.runAsUser -}}
{{- $_ := required "securityContext.fsGroup is required when setting securityContext and enabled is true" .Values.securityContext.fsGroup -}}
{{- if int .Values.securityContext.runAsUser | eq 0 -}}
{{- fail (dict "errorKey" "securityContext.runAsUser" "errorMessage" "Containers cannot be run as root users. \nPlease provide a user id greater than 0." | toJson) -}}
{{- end -}}
{{- end -}}
```
The `fail` function will always throw a fail message. In this case we are returning a json object to be parsed.


## Using `with`
This sets the value of the `.` variable. So you can just call the values out without typing the full key.
```
{{ with .Values.securityContext }}
          {{- if .enabled -}}
          securityContext:
            runAsUser: {{ .runAsUser }}
            fsGroup: {{ .fsGroup }}
          {{- end -}}
          {{- end }}
```

### Note
- Files with an "_" preceding the name will not be rendered in the `helm template` command.
- In Kubernetes you cannot have strings that are longer than 63 characters so it's common to truncate the string output.
- Add indentations to always ensure proper yaml or you're going to have issues once helm runs.

When iterating through a list using a `range` function, use `$` instead of `.` within the loop. This is because the meaning of `.` changes as you iterate down. 
The services.yaml looks like this:
```
services:
  - type: ClusterIP
    port: 80
  - type: NodePort
    port: 3007
```

This is arranged in the format of a list. The following code iterates through this list:
```
{{- range $idx, $svc := (.Values.services | default list) }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ include "templating-deep-dive.fullname" $ }}-{{ $idx }}
  labels: {{ include "templating-deep-dive.selectorLabels" $ | nindent 4 }}
spec:
  type: {{ $svc.type }}
  selector: {{ include "templating-deep-dive.selectorLabels" $ | nindent 4 }}
  ports:
    - protocol: TCP
      port: {{ $svc.port }}
      targetPort: {{ $.Values.containerPorts.http }}
{{- end }}
```

Adding the $idx in the Service name ensures that different services have unique names. But this becomes problematic when we change the order of the services.

The better way to do it is to refactor values.yaml like so:
```
services:
  svc1:  
    type: ClusterIP
    port: 80
  svc2:
    type: NodePort
    port: 30007
```

And then iterate through a dictionary instead:
```
{{- range $key, $svc := (.Values.services | default dict) }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ include "templating-deep-dive.fullname" $ }}-{{ $key }}
  labels: {{ include "templating-deep-dive.selectorLabels" $ | nindent 4 }}
spec:
  type: {{ $svc.type }}
  selector: {{ include "templating-deep-dive.selectorLabels" $ | nindent 4 }}
  ports:
    - protocol: TCP
      port: {{ $svc.port }}
      targetPort: {{ $.Values.containerPorts.http }}
{{- end }}
```
