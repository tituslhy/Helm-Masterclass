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

### Note
- Files with an "_" preceding the name will not be rendered in the `helm template` command.
- In Kubernetes you cannot have strings that are longer than 63 characters so it's common to truncate the string output.
- Add indentations to always ensure proper yaml or you're going to have issues once helm runs.