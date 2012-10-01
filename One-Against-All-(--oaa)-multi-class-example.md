## Overview
OAA stands for "One Against All".

### Purpose:
The option `--oaa <K>` where <K> is the number of distinct classes
directs vw to perform K multi-class (as opposed to binary) classification.

### Note:
* Data-set labels must be in the set {1 .. \<K\>}
* \<K\> is the maximum label value, and must be passed as an argument to `--oaa` 
                                                                  
### Implementation of the reduction:                                     
* Uses a loop of K separate binary classifications.                
* Each iteration classifies target feature i of K against all others (binary classification).

### example

Assume we have a 3-class classfication problem. We label our 3 classes {1,2,3}

Our data set may look like this

    $ cat oaa.dat
    1 ex1| a
    2 ex2| a b
    3 ex3| a b c
    2 ex4| b a
    1 ex5| d

We train:

    vw --oaa 3 oaa.dat -f oaa.model


And predict:

    vw -t -i oaa.model oaa.dat -p oaa.predict

