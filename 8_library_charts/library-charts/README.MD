# Library charts
Library charts are often used to define a common set of utility functions to be used by other charts.

Library charts typically inherit all its values from the parent chart and don't really have many yaml files in the templates folder.

Make sure to change the `type` argument in Chart.yaml of the library chart to `library`.

In this case, we:
- Created a subchart inside `parent-chart/charts` using `helm create common`
- Deleted everything in `parent-chart/charts/common/templates`
- Changed the `type` argument of `parent-chart/charts/Chart.yaml` to `library`
- Add `_naming.tpl` and copied and pasted the name and fullName definitions of parent chart's `_helpers.tpl` file, and changed `parent-chart.fullName`/`parent-chart.Name` to `common.fullName`/`common.Name`.
- Added `common` as a dependency to the parent chart
- Ran `helm dependency update <folder-of-parent-chart>` to get the tar file of the common helm chart.
- Refactored `deployment.yaml`, `service.yaml` and `_helpers.tpl` to call `common.fullName`/`common.Name` instead of `parent-chart.fullName`/`parent-chart.Name`.

The common sub chart therefore provided a "name" that was commonly used by its parent. 

Library charts are NOT expected to render any kubernetes resources