# Misc modeling projects in R

<style>
r { color: Red }
o { color: Orange }
g { color: Green }
</style>

Every folder contains separate project. Every folder contains `MarkDown` file with presentation of project.

These were part of university module thus the polish language.

Some of them require installation of `keras` to work. `R` says it prefers local anaconda installation and defaults to creating new one, which might not work out of the box. If you are using local virtualized environment commands:
```
$ tensorflow::install_tensorflow()
```
```
$ install_keras()
```
should do the job, but to be sure one might check e.g. https://tensorflow.rstudio.com/reference/keras/install_keras.
Short descriptions of projects

- `neu_net_mnist_conv` - Neural networks for number recognition on `mnist`. Tried to fit the model to recognize images with inverted colors using convolutional network. <r>Requires keras installation</r>
- `test_img_net` - Testing some existing image recognition networks: `resnet50`, `vgg16`, `vgg19`. <r>Requires keras installation</r>
- `workflowsets` - Example usage of workflow sets. Less flexible setup but code is cleaner. Cannot be integrated with keras.
- `klastrowanie` - Testing and comparing some `unsupervised learning` methods for clasification on mammals milk data e.g. k-means, hierarchical
- `klasyfikacja` - Testing and comparing some `classification` methods on banknotes dataset e.g. random forest or logistical regression
- `regresja` - Testing and comparing some `regression` methods on car-seats dataset e.g. linear, polynomial, tree