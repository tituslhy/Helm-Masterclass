# Intro-go-templating
This is a sandbox helm chart to show the impacts of Go templating. The Chart.yaml is very barebones and the `sandbox.yaml` file in the `templates` folder can be named anything.

Run the following from the `2_intro-go-templating` folder:
```
helm template .
```
and you'll notice that the expression is interpolated.

"Go" comments are not displayed, but yaml comments (comments with a "#" in front) will still be shown

## Using functions
In the `sandbox.yaml` file, the common way to write a function is:
```
{{- /* <function name> <arg1> <arg2> ...*/}}
```

Baseline:
```
test: {{ .Values.test }}
```

Lower case:
```
test: {{ lower .Values.test }}
```

Replace spaces with "-":
```
test: {{ replace " " "-" .Values.test }}
```

Chain functions:
```
test: {{ lower (replace " " "-" .Values.test) }}
```

Using the pipe operator in lieu of chaining:
```
test: {{ replace " " "-" .Values.test | lower }}
```

The results of the `replace` function is passed into `lower`

### Note
Conditional statement blocks must always have an `{{ end }}`