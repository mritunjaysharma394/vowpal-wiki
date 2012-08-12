I integrated murmur3 as a compile time (-DMURMUR3) option to vw.

## Results:

### Speed:
murmur3 brings a slight ~3% speed advantage in training time vs murmur2.

### Collision avoidance / dispersion
I used the data-sets in the test suite.  Most data-sets don't have collisions with neither hash.

I used --exact_adaptive_norm and varied the -b bits parameter from as low as 12 to stress the hash and induce collisions.

I found that on most data-sets I tried, murmur3 had a slight advantage in hash collision avoidance.

One exception was 0002.dat where murmur2 achieves no-colisions @ -b 18 (the default) while murmur3 achieves no-collisions only at -b 21.  The 2 colliding feature-names were short.

    Feature         Hash Value
    T^BKF                10281
    T^IWR                10281
    T^DXJ               100053
    T^GML               100053

Note: changing the seed value (hash_base constant in hash.h) from 97562527 to 0 avoids these 2 collisions. In the final commited version I switched to seed 0.  Most of the results below are with seed 97562527.

### Data-sets tested + number of features + winner-hash-method

                                      #-of-collisons
                                   @ -b 18 (default)    Winning hash method
    DataSet           #-features          M2      M3    (dominates most -b 12..30 range)
    0001.dat                4290           0       0    Murmur3
    0002.dat                 289           0       2    Murmur2
    zero.dat                1032           0       0    same
    wiki1K.dat              6330           0       0    same
    rcv1_small.dat         23530           1       0    Murmur3
    wsj_small.dat          13762         383     334    Murmur3
    ner.train             292497      116204  116108    Murmur3

### Random feature interaction as a result of hash collisions

Having less collisions doesn't always lead to lower training errors.
I found some evidence among our test-suite examples that random collisions may sometime act as some kind of regularization and the surprising result is a lower average loss at the end.

### Sample charts

Note, some charts have clipped X and/or Y ranges since otherwise the difference is too small to easily see.
I did run all test on all the 12 .. 30 bit range.

Also, not all charts have the same comparison criterion. Criterion was chosen to emphasize difference.

[[http://yendor.com/Img/wsj-m2-vs-m3-colpct.png]]

[[http://yendor.com/Img/ner-m2-vs-m3.png]]

[[http://yendor.com/Img/rcv1-m2-vs-m4-colpct.png]]

[[http://yendor.com/Img/ner-m3-vs-m2-pct-improvement.png]]




