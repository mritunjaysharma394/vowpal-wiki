## Setup
Start vw in active learning mode and listening on a port together with all the other options you need
`vw --active_learning --port 6075`

## Invoking active_interactor.py
active_interactor.py talks to vw in plain text via a TCP socket. It needs to know where vw is listening on and a set of unlabeled examples. These should be in text VW format except that the label should be missing. Suppose vw is running on the same machine, invoked as above, and the unlabeled data is in file unlabeled.dat. Then
`python active_interactor.py localhost 6075 unlabeled.dat` 
will connect to vw and start sending unlabeled examples in order. For each example that vw wants labeled, the user can specify the label (0,1) or 'skip' in which case no label is provided and it's (almost) as if the unlabeled dataset did not contain the example.
