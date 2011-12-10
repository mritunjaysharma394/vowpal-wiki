<h2>Truncated Gradient Descent Example for <a href="http://hunch.net/~vw/">VW</a></h2>

VW has an efficient (approximate) implementation of the <a href="http://jmlr.csail.mit.edu/papers/v10/chen09a.html">truncated gradient</a> algorithm for online L1 regularization.  This paper provides an example using the rcv1 data set to illustrate the use of it.  The (exact) online L2 regularization in VW can be done similarly, with the --l1 option below replaced by --l2.

We use the same training and test data prepared as in the [[RCV1 example]]; the cache files are cache_train and cache_test.  The test label file will be needed for classifier evaluation, and is obtained by
<pre>
zcat rcv1.test.dat.gz | cut -d ' ' -f 1 | sed -e 's/^-1/0/' > test_labels
</pre>

The following three steps run (1) training, (2) testing, (3) evaluation of ROC, and (4) measuring model size, respectively:
<pre>
vw --cache_file cache_train --final_regressor r_temp --passes 3 --readable_model r_temp.txt --l1 lambda1
vw --testonly --initial_regressor r_temp --cache_file cache_test --predictions p_out
perf -ROC -files test_labels p_out
cat r_temp.txt | grep -c ^[0-9]
</pre>
where
<ol>
<li>lambda1 is the regularization level applied to online learning</li>
<li>r_temp.txt is the human-readable model file for us to count the number of nonzero weights in the learned regressor</li>
</ol>

By varying lambda1, we see the role of L1 regularization on prediction performance (ROC in particular) and model size:
<table border=1>
<tr><td>lambda1</td><td>ROC</td><td>Model Size</td></tr>
<tr><td>0</td><td>0.98346</td><td>41409</td></tr>
<tr><td>5e-8</td><td>0.98345</td><td>39985</td></tr>
<tr><td>1e-7</td><td>0.98345</td><td>38822</td></tr>
<tr><td>5e-7</td><td>0.98345</td><td>31899</td></tr>
<tr><td>1e-6</td><td>0.98345</td><td>26559</td></tr>
<tr><td>5e-6</td><td>0.98319</td><td>12564</td></tr>
<tr><td>1e-5</td><td>0.98288</td><td>7647</td></tr>
<tr><td>5e-5</td><td>0.98068</td><td>1860</td></tr>
<tr><td>1e-4</td><td>0.97804</td><td>921</td></tr>
<tr><td>1e-3</td><td>0.92469</td><td>53</td></tr>
</table>

Note that L1 and L2 can be used simultaneously in VW, which resembles the elastic net.  To see the role of L2-regularization better, the training data is first subsampled at 1% rate, yielding a set of roughly 7.8K examples.  Let cache_train_small be the training cache file.  The previous commands are modified slightly by adding the --l2 option as follows:
<pre>
vw --cache_file cache_train_small --final_regressor r_temp --passes 1000 --readable_model r_temp.txt --l1 lambda1 --l2 lambda2
vw --testonly --initial_regressor r_temp --cache_file cache_test --predictions p_out
perf -ROC -files test_labels p_out
cat r_temp.txt | grep -c ^[0-9]
</pre>
Note that we set the number of passes to 1000 so that we can see the phenomenon of overfitting.

The table below reports the ROC metric and model size by varying lambda1 and lambda2:
<table border=1>
<tr><td>lambda1</td><td>lambda2</td><td>ROC</td><td>Model Size</td></tr>
<tr><td>0</td><td>0</td><td>0.96863</td><td>20832</td></tr>
<tr><td>0</td><td>0.0005</td><td>0.97364</td><td>21490</td></tr>
<tr><td>1e-7</td><td>0.0005</td><td>0.97364</td><td>21470</td></tr>
<tr><td>1e-6</td><td>0.0005</td><td>0.97363</td><td>21149</td></tr>
<tr><td>1e-5</td><td>0.0005</td><td>0.97348</td><td>14857</td></tr>
<tr><td>5e-5</td><td>0.0005</td><td>0.97231</td><td>4185</td></tr>
<tr><td>1e-4</td><td>0.0005</td><td>0.97003</td><td>2020</td></tr>
</table>



