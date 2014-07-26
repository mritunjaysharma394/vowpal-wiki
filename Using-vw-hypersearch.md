## Introduction

`vw-hypersearch` is a simple wrapper to `vw` to help in finding lowest-loss hyper-parameters *_(argmin)_*.

For example: say you want to find the lowest average loss for `--l1` (L1-norm regularization) on a train-set called `train.dat`.  You can run:
```
    $ vw-hypersearch 1e-10 1 vw --l1 % train.dat
```

`vw-hypersearch` will train multiple times (but in a efficient way) until it finds the `--l1` value resulting in the lowest average training loss.

In the call:

* the `%` character is a placeholder for the (argmin) parameter we are looking for.
* `1e-10` is the lower-bound for the search range
* `1` is the upper-bound of the search range

The lower & upper bounds are arguments to `vw-hypersearch`. Anything from `vw` on, are normal `vw` arguments exactly as you would use in training.  The only change you must apply to the training command is to use `%` instead of the *_value_* of the parameter you're trying to optimize on.

Calling `vw-hypersearch` without any arguments should provide a Usage message.

## More advanced options

Additional arguments can be passed to `vw-hypersearch` preceding `vw` itself:

* -L will do a _log-space_ golden-section search instead of a _simple_ golden-section search.
* `-t test.dat` (note: this must come _before_ the `vw` argument) will search for the training parameter that results in a minimum loss on `test.dat` rather than `train.dat` (ignoring the training errors). 
* An optional 3rd numeric parameter will be interpreted as a `tolerance` parameter directing `vw-hypersearch` to stop only when a difference in two consecutive run errors is less than `tolerance`


## Examples

    # Find the learning-rate resulting in the lowest average loss
    # for a logistic loss train-set:
    vw-hypersearch 0.1 100 vw --loss_function logistic --learning_rate % train.dat

    # Find the bootstrap resulting in the lowest average loss
    # vw-hypersearch will automatically search in integer-space
    # since --bootstrap expects an integer
    vw-hypersearch 2 16 vw --bootstrap % train.dat

## Implementation notes

`vw-hypersearch` conducts a [golden-section search](http://en.wikipedia.org/wiki/Golden_section_search) search by default.  This search method strikes a good balance between safety and efficiency.
 
## Caveats

* Lowest average loss is not necessarily optimal
* Your real goal should always be to find a minimal generalization error, not training error.
* Some parameters do not have a convex loss, for these `vw-hypersearch` will converge on some local-minimum instead of a global one

## More questions?

`vw-hypersearch` is written in perl and is included with vowpal wabbit (in the `utl` subdirectory). In case of doubt, look at the source