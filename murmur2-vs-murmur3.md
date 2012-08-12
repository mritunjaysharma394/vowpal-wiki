I integrated murmur3 as a compile time (-DMURMUR3) option to vw.

## Results:

### Speed:
murmur3 brings a slight ~3% speed advantage in training time vs murmur2.

### Collision avoidance / dispersion
I used the data-sets in the test suite.  Most data-sets don't have collisions with neither hash.

I used --exact_adaptive_norm and varied the -b bits parameter from as low as 12 to stress the hash and induce collisions.

I found that on most data-sets I tried, murmur3 had a slight advantage in hash collision avoidance.

One exception was 0002.dat where murmur2 achieves no-colisions @ -b 18 (the default) while murmur3 achieves no-collisions only at -b 21.

### Data-sets tested with number of features


    DataSet           #-features     Winner (dominates on most -b bits range)
    0001.dat                4290     Murmur3
    0002.dat                 289     Murmur2
    rcv1_small.dat         23530     Murmur3
    wsj_small.dat          13762     Murmur3
    ner.train             292497     Murmur3

### Charts

[[http://yendor.com/Img/wsj-m2-vs-m3-colpct.png]]

[[http://yendor.com/Img/rcv1-m2-vs-m4-colpct.png]]

[[http://yendor.com/Img/ner-m3-vs-m2-pct-improvement.png]]

[[http://yendor.com/Img/ner-m2-vs-m3.png]

