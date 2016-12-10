## Overview
ECT stands for "Error Correcting Tournament".

### Purpose:
The option `--ect <K>` where \<K\> is the number of distinct classes
directs vw to perform K multi-class (as opposed to binary)
classification.  This methods differs from `--oaa <K>` only in
the algorithm used.  The data input format is the same.

### Note:
* Data-set labels must be in the natural number set {1 .. \<K\>}
* \<K\> is the maximum label value, and must be passed as an argument to `--ect`

### Implementation of the reduction:                                     
* See the paper http://arxiv.org/abs/0902.3176

### Example

Assume we have a 3-class classification problem. We label our 3 classes {1,2,3}

Our data set `ect.dat` may look like this

    1 ex1| a
    2 ex2| a b
    3 ex3| c d e
    2 ex4| b a
    1 ex5| f g

This is essentially the same format as the non multi-class case
(classification or regression) where each label must belong to one
of the {1..\<K\>} classes, i.e. a natural number between 1 and
\<K\>. You may add weights to the example and the features, use
name-spaces, etc.

We train:

    vw --ect 3 ect.dat -f ect.model

Which gives this progress output:
```
final_regressor = ect.model
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
using no cache
Reading datafile = ect.dat
num sources = 1
average  since         example        example  current  current  current
loss     last          counter         weight    label  predict features
0.000000 0.000000            1            1.0        1        1        2
0.500000 1.000000            2            2.0        2        1        3
0.500000 0.500000            4            4.0        2        2        3

finished run
number of examples per pass = 5
passes used = 1
weighted example sum = 5.000000
weighted label sum = 0.000000
average loss = 0.600000
total feature number = 15
```

Now we can predict, using the same data set as our test-set:

    vw -t -i ect.model ect.dat -p ect.predict

Similar to what we do in vanilla classification or regression.
The resulting `ect.predict` file has contents:
```
2.000000 ex1
2.000000 ex2
3.000000 ex3
2.000000 ex4
1.000000 ex5
```
Which is as expected: 'ex5' belong to class 1, 'ex1', 'ex2' and
'ex4' belong to class 2, and 'ex3' belongs to class 3.
