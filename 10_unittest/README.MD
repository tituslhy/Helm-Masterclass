# Unit testing in Helm
Unit tests in helm are a plugin maintained by the community, and are typically used if there is logic coded into your helm chart.

To install the plugin:
```
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

Conventionally, you have to create a tests/ folder in your root folder, then write your unit tests in the yaml files within that folder. It's also suggested to add this tests folder to your `.helmignore` file. 

Unit tests are typically placed in a list format:
```
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should match the naming convention of ending in "my-chart"    
    asserts:      
      - matchRegex:
          path: metadata.name
          pattern: -my-chart$      
  - it: should correctly use the "image.tag" property when set
    set:
      image.tag: latest
    asserts:
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:latest  
```
There are two tests here that verify if the name of the chart has a "my-chart" suffix, and if the image.tag property is used correctly.

To run unittests:
```
helm unittest <folder-of-helm-chart>
```

To make it verbose, add a `-d` tag. You will be able to see debug level logs.

### Note
These unit tests are different from the test hooks inside `./templates/tests/`

All test.yaml files used for unittesting MUST END WITH `_test.yaml`

The unit test plugin does not create any kubernetes resources at all!