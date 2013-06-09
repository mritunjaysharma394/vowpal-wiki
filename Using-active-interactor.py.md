## Setup
Start vw in active learning mode and listening on a port together with all the other options you need

`vw --active_learning --port 6075`

## Invoking active_interactor.py
active_interactor.py talks to vw in plain text via a TCP socket. It needs to know where vw is listening on and a set of unlabeled examples. These should be in text VW format except that the label should be missing. Suppose vw is running on the same machine, invoked as above, and the unlabeled data is in file unlabeled.dat. Then

`python active_interactor.py localhost 6075 unlabeled.dat` 

will connect to vw and start sending unlabeled examples in order. For each example that vw wants labeled, the user can specify the label (0,1) or 'skip' in which case no label is provided and it's (almost) as if the unlabeled dataset did not contain the example.

## Options
The following options are currently supported  by active_interactor.py
* -s labeled_set. Uses the labeled examples in labeled_set to seed vw's model prior to sending any unlabeled examples.
* -v. Verbose mode. Prints the unlabeled example before asking for the label (otherwise only a line number and a tag are used). This makes sense for text classification with data in close to raw form.
* -m. Interprets a 0 as -1
* -o output. Writes the user provided labels (and corresponding examples) to a (line buffered) file. The absence of full buffering allows one to hit Ctrl-C anytime and still have all their labels saved in the output file.

## Tips
Choice of hyperparameters is tricky. If you have a labeled subsample, use it together with `--active_simulation` to discover a good value for `--active_mellowness` and the learning rate.