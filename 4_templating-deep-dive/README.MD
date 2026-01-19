# Learning Go Templates

## Template functions and pipelines
Visit [Helm documentation on functions](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)

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