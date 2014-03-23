# Running vw in daemon mode

**Use case:** You have a model and you want `vw` to efficiently serve predictions based on the model.

When you run `vw` in daemon mode.  It loads the model once into memory at startup (using `-i modelfile`) and listens to requests coming over a tcp socket (using the `--daemon` option). If you run `vw` in test-only (`-t`) mode, it will only test (i.e. predict).  Every write of an example into the socket should result in an immediate response on the same socket.

In performance tests I (arielf) was able to sustain a throughput of about 50,000 requests+predictions per second on standard hardware on a simple (~20 feature) model. 

**Deployment note:** since `vw` is already highly optimized for performance, the step of rewriting code to deploy a model in production is redundant. Using `vw` itself for deployment, with the same model that was produced during training, is the recommended way to deploy a model.

Here's a short howto:

### Training set `train.vw`
Our little training set `train.vw` has only two examples.  The features `a`, `b`, and `c` have a `0` label, and the features `x`, `y`, and `z` have a `1` label:

```
$ cat train.vw
0 example0| a b c
1 example1| x y z
```

### Training

Due to the tiny number of examples in the training-set, we use 20 passes to achieve perfect convergence (0 progressive loss). We run a (default) squared-loss regression.
```
$ vw -c --passes 20 --holdout_off train.vw -f model.vw
final_regressor = model.vw
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
decay_learning_rate = 1
using cache_file = train.vw.cache
num sources = 1
average    since         example     example  current  current  current
loss       last          counter      weight    label  predict features
0.000000   0.000000            1         1.0   0.0000   0.0000        4
0.500000   1.000000            2         2.0   1.0000   0.0000        4
0.275101   0.050201            4         4.0   1.0000   0.7458        4
0.138850   0.002599            8         8.0   1.0000   0.9692        4
0.069436   0.000022           16        16.0   1.0000   0.9994        4
0.034718   0.000000           32        32.0   1.0000   1.0000        4

finished run
number of examples per pass = 2
passes used = 20
weighted example sum = 40
weighted label sum = 20
average loss = 0.0277744
best constant = 0.5
best constant's loss = 0.25
total feature number = 160
```

### Starting the daemon

Now that we have a model `model.vw`, we can start `vw` in daemon mode:

```
$ vw -i model.vw  -t --daemon --quiet --port 26542

# Check that the daemon is running:
# By default we have one parent and 10 children processes
# This number of children (worker threads) can be changed with --num_children <N>

$ pgrep vw| wc -l
11

```

### Sending examples and getting predictions

```
$ echo " abc-example| a b c" | netcat localhost 26542
0.000000 abc-example

$ echo " xyz-example| x y z" | netcat localhost 26542
1.000000 xyz-example
```
As expected, we got a prediction of `0` for `a b c` and a prediction of `1` for `x y z`.
The tags on the two examples are for illustrative purposes, they aren't mandatory.

There's no need to read the response after every request.  Since network buffers are large, you may also send multiple examples and then read multiple responses.  A new line character is both the input and the output record (example) separator.
```
$ echo '| a c
| b c
| y x
| z y x' | netcat localhost 26542
0.079382
0.079382
0.746049
1.000000
```

### Stopping the daemon
```
$ killall vw

# Verify that it no longer runs:
$ pgrep vw | wc -l
0
```
