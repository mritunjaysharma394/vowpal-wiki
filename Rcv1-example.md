<h2>RCV1-V2 Example for <a href="http://hunch.net/~vw/">VW</a></h2>
One source for test data come from <a href="http://leon.bottou.org/">Leon Bottou</a>'s page on <a href="http://leon.bottou.org/projects/sgd">stochastic gradient descent</a>.  To use this, download the RCV1-V2 dataset <a href="http://hunch.net/~vw/rcv1.tar.gz">here</a>. <p>

There are 3 files, rcv1.train.dat.gz, rcv1.test.dat.gz and vw_process.  vw_process is a simple script that converts from an svmlight format to VW's format.

The individual files look like:
<pre>
1 |f 13:3.9656971e-02 24:3.4781646e-02 69:4.6296168e-02 85:6.1853945e-02 ... 
0 |f 9:8.5609287e-02 14:2.9904654e-02 19:6.1031535e-02 20:2.1757640e-02 ... 
...
</pre>
From the above, you can see that the input data format is similar to <a href="http://svmlight.joachims.org/">SVMlight</a>'s feature:value sparse representation format.  There are two important differences: 
<ol>
<li>The feature can be a string not including colon ':', space ' ', or pipe '|' (which are special characters).  In the above, this is not used.  In general, this is pretty handy because you can use much less processed data than most learning algorithm take in.</li>
<li>The features are divided into namespaces.  The semantics of a namespace is: features with the same name but a different namespace are different features.  This example just has one namespace "f" which is the simplest (and probably most common) case.</li>
</ol>

There are a couple variation on the above format.  If you want to importance weight examples, place the importance weight after the label and before the first namespace.  A missing importance weight is treated as <em>1</em> by default.  Similarly, if features have a weight of <em>1</em>, they can be represented as just it's name rather than name:1.

<p>
A command for training is the following.
<pre>
zcat rcv1.train.dat.gz | vw --cache_file cache_train -f r_temp 
</pre>
Here: 
<ol>
<li><strong>--cache_file</strong> flag parses the data into VWs own internal compressed format.  The second time you run the above command, it should be much faster---about 1.5 seconds on my current desktop machine.</li>
<li><strong>-f r_temp</strong> stores the output regressor in the file r_temp.</li>
</ol>

Next, you can test according to the following:
<pre>
zcat rcv1.test.dat.gz | vw -t --cache_file cache_test -i r_temp -p p_out
</pre>
Here the flags are:
<ol>
<li> <strong>-t</strong> tells vw to not use the labels for training.</li>
<li><strong>-i r_temp</strong> loads the regressor at r_temp before examples are processed</li>
<li> <strong>-p p_out</strong> makes the predictions be output to the file p_out.</li>
</ol>
To measure performance, I often use the <a href="http://osmot.cs.cornell.edu/kddcup/software.html">perf</a> which Rich Caruana put together for the 2004 KDD cup challenge.  This software has the advantage that many people cared that it worked right.  To use perf, you first create a file with the labels 
<pre>
zless rcv1.test.dat.gz | cut -d ' ' -f 1 | sed -e 's/^-1/0/' > labels
</pre>

and then type:
<pre>
perf -ACC -files labels p_out -t 0.5
</pre>

The results on my machine are summarized by the following table(*): 
<table border=1>
<tr><td>Method</td><td>Wall clock Execution Time</td><td>Test Set Error rate</td></tr>
<tr><td>VW</td><td>3.0s</td><td>5.54%</td></tr>
<tr><td>svmsgd</td><td>21.8s</td><td>5.74%</td></tr>

</table>
There are several things to understand about the results.
<ol>
<li>VW is optimizing squared loss and then thresholding on 0.5 rather than hinge loss, so their internal optimizations fundamentally differ.  For svmsgd, 5.74% was the best I found for one pass with lambda = 0.00001.  Note that nobody is worrying about overfitting in parameter tuning here.</li>
<li>The timing numbers are wall-clock execution time.  svmsgd spends about 19s (wall clock time) loading the data and making it ready to train in 0.38s (cputime).  VW runs fully online, so the process of loading and running data is fundamentally mixed.</li>
</ol>

(*) The comparison with svmsgd is obsolete, as Leon has updated his code.