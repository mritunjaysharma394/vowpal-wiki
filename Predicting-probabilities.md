## Binary classification
By default, VW predictions are in the range `[-50, +50]` (theoretically any real number, but VW truncates it to [-50, +50]).
To convert them to {-1, +1}, use `--binary` – positive predictions are simply mapped to +1, negative to -1.
To convert them to [0, +1], use `--link=logistic`. This uses the logistic function 1/(1 + exp(-x)). If you use `--loss_function=logistic`, you can interpret the numbers as probabilities of the +1 class. So to output the probability prediction to a file **use `--loss_function=logistic --link=logistic -p probs.txt`**.

(For completeness, to convert the predictions to [-1, +1], use `--link=glf1`. This uses formula 2/(1 + exp(-x)) - 1, i.e. generalized logistic function with limits of 1.)

## Multi-class `--oaa`
By default in [oaa](One-Against-All-%28oaa%29-multi-class-example), VW predicts the label of the most probable class.
To get probabilities for each of the 1..N classes, simply **use `--oaa N --loss_function=logistic --probabilities -p probs.txt`**.
Logistic link function will be used automatically (you should not specify it with `--link`) and the numbers (probabilities) will be normalized so they sum up to 1.
See example usage in [test 109](https://github.com/JohnLangford/vowpal_wabbit/blob/41befcc1fb86c4aaf5e96f91c0cf4427218ea4fe/test/RunTests#L1357).

When using `--probabilities` during training (or testing with gold costs available), a multi-class logistic loss is reported in addition to the standard zero-one loss.
The option `--probabilities` is stored in the model, so it does not need (actually cannot) be repeated when testing.

## Multi-class `csoaa_ldf`
For [cost-sensitive oaa with label-dependent features](http://www.umiacs.umd.edu/~hal/tmp/multiclassVW.html), 
**use `--csoaa_ldf=mc --loss_function=logistic --probabilities -p probs.txt`**.
See example usage in [test 110](https://github.com/JohnLangford/vowpal_wabbit/blob/41befcc1fb86c4aaf5e96f91c0cf4427218ea4fe/test/RunTests#L1362).

## TODO
Add command-line examples here (with input data and expected output) instead of linking tests.