## Introduction

`vw-hypersearch` is a simple wrapper to `vw` to help in finding lowest-loss hyper-parameters *_(argmin)_*.

For example: to find the lowest average loss for `--l1` (L1-norm regularization) on a train-set called `train.dat`:
```
    $ vw-hypersearch  1e-10  5e-4  vw  --l1 %  train.dat
```

`vw-hypersearch` will train multiple times (but in a efficient way) until it finds the `--l1` value resulting in the lowest average training loss.

Explanation of the example:

* the `%` character is a placeholder for the (argmin) parameter we are looking for.
* `1e-10` is the lower-bound for the search range
* `5e-4` is the upper-bound of the search range

The lower & upper bounds are arguments to `vw-hypersearch`. Anything from `vw` on, are normal `vw` arguments exactly as you would use in training.  The only change you must apply to the training command is to use `%` instead of the *_value_* of the parameter you're trying to optimize on.

Calling `vw-hypersearch` without any arguments prints a Usage message.

## More advanced options

Additional arguments can be passed to `vw-hypersearch` preceding `vw` itself:

* -L will do a _log-space_ golden-section search instead of a _simple_ golden-section search.
* `-t test.dat` (note: this must come _before_ the `vw` argument) will search for the training parameter that results in a minimum loss on `test.dat` rather than `train.dat` (ignoring the training errors)
* `-c test.dat.cache` use `test.dat.cache` as test-cache file + evaluate goodness on it (this implies `-t` except the cache `test.dat.cache` will be used instead of `test.dat`)
* `-e <script_name>` or even `-e '<script_name> <script_args>...'` will use an external program `<script_name>` and will try to minimize its last numeric output. This allows you to plug-in any external tool for the `argmin`.  For example, you could write a script that will call `vw` with the just generated model, in prediction mode, and then use `perf` to calculate accuracy, various AUCs etc. The only interface between `vw-hyperseach` and the <script_name> is the last numeric value printed to `stdout` by <script_name> which vw-hypersearch will try to minimize.  Negate the value inside <script_name> if you'd rather maximize the value. 
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

`vw-hypersearch` is written in perl and is included with vowpal wabbit (in the `utl` subdirectory). In case of doubt, look at the source.

## Credits
```
#   * Paul Mineiro ('vowpalwabbit/demo') wrote the original script
#   * Ariel Faigon
#       - Generalize, rewrite, refactor
#       - Add usage message
#       - Add tolerance as optional parameter
#       - Add documentation
#       - Better error messages when underlying command fails for any reason
#       - Add '-t test-set' to optimize on test error rather than
#         train-set error
#       - Add integral-value option support
#       - More reliable/informative progress indication
#       Bug fixes:
#           - Golden-section search bug-fix
#           - Loss value capture bug-fix (can be in scientific notation)
#           - Handle special cases where certain options don't make sense
#   * Alex Hudek:
#       - Support log-space search which seems to work better
#         with --l1 (very small values) and/or hinge-loss
#   * Alex Trufanov:
#       - A bunch of very useful bug reports:
#           https://github.com/JohnLangford/vowpal_wabbit/issues/406
```