## Overview
CSOAA stands for "Cost Sensitive One Against All" - A multi-class predictive modeling reduction in VW.

### Purpose:
The option `--csoaa <K>` where \<K\> is the number of distinct classes
directs vw to perform cost-sensitive K multi-class (as opposed to binary)
classification.  It extends `--oaa <K>` to support multiple labels per input example, and costs associated with classifying these labels.

### Notes:
* Data-set labels must be in the natural number set {1 .. \<K\>}
* \<K\> is the maximum label value, and must be passed as an argument to `--csoaa`
* The input/training format for `--csoaa <K>` is different than the traditional VW format:
    * It supports *multiple labels* on the *same line*
    * Each label has a trailing *cost*
    * Cost syntax *looks* just like weight syntax: a colon followed by a floating-point number.
      For example:  `4:3.2` means the class-label 4 with a cost of 3.2, but *means* the opposite of weights.
    * It is critical to note that costs are *not* weights. They are the inverse of weights.
      A label with a *lower* cost is prefered over a label with a higher cost on the same line.
      That's why they are called `'costs'`.
* The reduction with `--csoaa` is to a regression problem (i.e. conditional mean estimation), so forcing the loss function to logistic does not make much sense. Generally, when using multi-class, you should leave the `--loss_function` alone and let the algorithm use the built-in default. 

### Example
Assume we have a 3-class classfication problem. We label our 3 classes {1,2,3}

Our data set `csoaa.dat` is:

    1:1.0 a1_expect_1| a
    2:1.0 b1_expect_2| b
    3:1.0 c1_expect_3| c
    1:2.0 2:1.0 ab1_expect_2| a b
    2:1.0 3:3.0 bc1_expect_2| b c
    1:3.0 3:1.0 ac1_expect_3| a c
    2:3.0 d1_expect_2| d

Notes:
* The first 3 examples (lines) have only one label (with costs) each, and the next 3 examples have multiple labels on the same line. Any number of class-labels between {1 .. \<K\>} (1..3 in this case) is allowed on each line.
* We assign a _lower_ cost to the label we want to be preferred. e.g. in line 4 (tagged `ab1_expect_2`) we have a cost of 1.0, for class-label 2; and a higher cost 2.0, for class-label 1.
* The input feature section following the '|' is the same as in  traditional VW: you may have multiple name-spaces, numeric features, and optional *weights* for features and/or name-spaces (Note in this section the weights are weights, not costs, so they are positively correlated with chosen labels)

We train:

    vw --csoaa 3 csoaa.dat -f csoaa.model

Which gives us this progress output:

    final_regressor = csoaa.model
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading from csoaa.dat
    num sources = 1
    average    since         example     example  current  current current
    loss       last          counter      weight    label  predict features
    0.000000   0.000000          3           3.0    known        3        2
    0.833333   1.666667          6           6.0    known        1        3

    finished run
    number of examples = 7
    weighted example sum = 7
    weighted label sum = 0
    average loss = 0.7143
    best constant = 0
    total feature number = 17

Now we can predict, loading the model `csoaa.model` and using the same data-set `csoaa.predict` as our test-set:

    vw -t -i csoaa.model csoaa.dat -p csoaa.predict

Similar to what we do in vanilla classification or regression.

The resulting `csoaa.predict` file has contents:

    1.000000 a1_expect_1
    2.000000 b1_expect_2
    3.000000 c1_expect_3
    2.000000 ab1_expect_2
    2.000000 bc1_expect_2
    3.000000 ac1_expect_3
    2.000000 d1_expect_2

Which is a perfect classification:

* all the `expect_1` lines have a predicted class of 1,
* all the `expect_2` lines have a predicted class of 2,
* and all the `expect_3` lines have a predicted class of 3.

QED

## Credits
* Thanks to Ciemo for the example and for asking the right Qs on the mailing list.
* Thanks to Stephane for patiently answering Ciemo's Qs.


