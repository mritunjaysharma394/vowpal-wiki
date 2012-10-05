## Overview
OAA stands for "One Against All".

### Purpose:
The option `--oaa <K>` where <K> is the number of distinct classes
directs vw to perform K multi-class (as opposed to binary) classification.

### Note:
* Data-set labels must be in the natural number set {1 .. \<K\>}
* \<K\> is the maximum label value, and must be passed as an argument to `--oaa` 
                                                                  
### Implementation of the reduction:                                     
* Uses a loop of K separate binary classifications.                
* Each iteration classifies target feature i of K against all others (binary classification).

### Example

Assume we have a 3-class classfication problem. We label our 3 classes {1,2,3}

Our data set `oaa.dat` may look like this

    1 ex1| a
    2 ex2| a b
    3 ex3| c d e
    2 ex4| b a
    1 ex5| f g

This is essentially the same format as the non multi-class case (classification or regression) where each label must belong to one of the {1..\<K\>} classes, i.e. a natural number between 1 and \<K\>. You may add weights to the example and the features, use name-spaces, etc.

We train:

    vw --oaa 3 oaa.dat -f oaa.model

Which gives this progress output:
    final_regressor = oaa.model
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading from oaa.dat
    num sources = 1
    average    since         example     example  current  current  current
    loss       last          counter      weight    label  predict features
    0.666667   0.666667          3      3.0          3        1        4

    finished run
    number of examples = 5
    weighted example sum = 5
    weighted label sum = 0
    average loss = 0.4
    best constant = 0
    total feature number = 15

We then can try and predict, using the same data set as out test-set:

    vw -t -i oaa.model oaa.dat -p oaa.predict

Similar to what we do in vanilla classification or regression.
The resulting `oaa.predict` file has contents:

    1.000000 ex1
    2.000000 ex2
    3.000000 ex3
    2.000000 ex4
    1.000000 ex5

Which is as expected: 'ex1' and 'ex5' belong to class 1, 'ex2' and 'ex4' belong to class 2, and 'ex3' belongs to class 3.
