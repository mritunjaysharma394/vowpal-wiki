[OjaNewton](http://arxiv.org/abs/1602.02202) is a sketched variant of a second order online learning algorithm called Online Newton Step [(ONS)](http://www.cs.princeton.edu/~ehazan/papers/log-journal.pdf). It overcomes the quadratic running time of ONS by performing the Oja's updates to keep a small sketch of the covariance matrix of gradients in a sparse manner.

### Example of using OjaNewton in VW:
```
vw  --OjaNewton --sketch_size=10 --alpha_inverse=1.0  -d train_set_file  -f trained_model_file
```
Here **sketch_size** is the number of directions that we keep for the covariance matrix (default is 10) and **alpha_inverse** can be viewed as a learning rate (default is 1.0).

Then, to predict from the trained model and a new data set:
```
vw -i trained_model_file -d data_set -p prediction_file
```