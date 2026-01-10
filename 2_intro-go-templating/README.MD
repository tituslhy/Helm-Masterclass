# Intro-go-templating
This is a sandbox helm chart to show the impacts of Go templating. The Chart.yaml is very barebones and the `sandbox.yaml` file in the `templates` folder can be named anything.

Run the following from the `2_intro-go-templating` folder:
```
helm template .
```
and you'll notice that the expression is interpolated.

"Go" comments are not displayed, but yaml comments (comments with a "#" in front) will still be shown