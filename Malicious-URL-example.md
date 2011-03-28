# Detecting Malicious URLs

The [malicious URL dataset from UCSD](http://www.sysnet.ucsd.edu/projects/url/) represents a sequential binary classification problem.  The data are temporally correlated, and thus the problem is particularly suitable for online learning approaches like VW.  Here we show how to evaluate the "out-of-the-box" performance of VW on this task.

## Preparing the data

First, download the [data in SVM-light format](http://www.sysnet.ucsd.edu/projects/url/url_svmlight.tar.gz) and extract the files from the tar-ball.
    tar xzf url_svmlight.tar.gz

The following command-line converts these data from SVM-light format to VW input format:
    for d in `seq 0 120`; do cat url_svmlight/Day$d.svm; done \
      |sed -e 's/^-1/0 |f/' |sed -e 's/^+1/1 |f/' |sed -e 's/$/ const:.01/'

This conversion accomplishes the following:

* Converts the labels from "-1" to "0" and from "+1" to "1"
* Puts the features into a namespace called "f"
* Adds a constant feature called "const" with value ".01".
* Retains the temporal order of the data.

This shell pipeline can be used directly with VW, as described below.

## Out-of-the-box online training and testing

We can use the above command-line to feed the data directly into VW to simulate online training and testing:
    time for d in `seq 0 120`; do cat url_svmlight/Day$d.svm; done \
      |sed -e 's/^-1/0 |f/' |sed -e 's/^+1/1 |f/' |sed -e 's/$/ const:.01/' \
      |vw --adaptive --cache_file cache

The [[command line arguments]] used above are:

* `--adaptive`: use per-feature adaptive learning rates; this is sensible for highly diverse and variable features
* `--cache_file cache`: cache the parsed input data into the file `cache`

It also uses `time` to measure the approximate wall-clock execution time.

### Results

The output of the above command-line concludes with the following.

    finished run
    number of examples = 2396130
    weighted example sum = 2.396e+06
    weighted label sum = 7.921e+05
    average loss = 0.0127
    best constant = 0.3306
    best constant's loss = 0.2213
    total feature number = 281850904
    
    real    3m28.111s
    user    2m36.850s
    sys     0m17.050s

The average square loss over all 2396130 examples is 0.0127.  The wall-clock execution time is 3 minutes 28 seconds.  This may alarm you (or not), but most of time is spent parsing.  If you re-run the command-line, it will read the cached data from `cache` and give the same result, except for the execution time:

    real    0m17.982s
    user    0m20.650s
    sys     0m9.340s

### Evaluating prediction accuracy

If you want to compare the actual predictions to the true labels, re-run the command-line with the additional option `--predictions p_out` to output the predictions to the file `p_out`.  Then extract the labels from the training data using the following command-line:

    for d in `seq 0 120`; do cat url_svmlight/Day$d.svm; done \
      |cut -d ' ' -f 1 |sed -e 's/^-1/0/' >labels

One can use Rich Caruana's [perf](http://kodiak.cs.cornell.edu/kddcup/software.html) software to compute the cumulative accuracy, but this requires a minor tweak in the code to allow more than 500000 predictions.  Once that is dealt with, executing the command-line:

    perf -ACC -files labels p_out -t 0.5

should give the result:

    ACC    0.98364   pred_thresh  0.500000

98.36% accuracy by thresholding predictions at 0.5.