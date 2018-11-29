The `--rank` option switches VW to matrix factorization mode for interaction terms (as given by `-q`). The argument to `--rank` specifies the rank of the interaction matrix (the number of latent factors) in the model. Simple stochastic gradient descent is performed to update linear and quadratic terms. In contrast to traditional matrix factorization, VW's hashing technique constrains the memory footprint of the learned model, enabling the system to scale to large data sets with many examples and features.

In this mode interaction terms are approximated by a low-rank matrix, which has shown good performance in [model-based collaborative filtering](http://en.wikipedia.org/wiki/Collaborative_filtering#Model-Based) for recommendation systems. See [Matrix Factorization Techniques for Recommender Systems](http://research.yahoo.com/files/ieeecomputer.pdf) by Koren, Bell & Volinksy for an introduction and overview on the topic.

For example, consider the problem of predicting movie ratings for the [Movielens dataset](http://www.grouplens.org/node/73) of 100K ratings: given a user and a movie (which has not yet been rated by that user), predict the rating that the user will assign the movie.

The ratings can be obtained as follows:

    wget http://www.grouplens.org/system/files/ml-100k.zip
    unzip ml-100k.zip
    cd ml-100k

The data consist of `(user, item, rating, date)` events, where ratings are given on an (integer) scale of 1 to 5. Using awk to reformat the data to a VW-friendly format, we can learn a model with a constant term (representing a global average), linear terms (representing per-user and per-item rating biases) and a rank-10 approximation to the interaction terms (representing user-item interactions) as follows:

    awk -F"\t" '{printf "%d |u %d |i %d\n", $3,$1,$2}' < ua.base | \
      ../build/vowpalwabbit/vw /dev/stdin -b 18 -q ui --rank 10 --l2 0.001 \
      --learning_rate 0.015 --passes 20 --decay_learning_rate 0.97 --power_t 0 \
      -f movielens.reg --cache_file movielens.cache

Note that the combination of `-b 18` and `--rank 10` results in a weight vector of (1+2*10)*2^18 elements. The `--l2` option is the L2 regularization argument to avoid overfitting. For reference, the first few lines of input sent to Vowpal Wabbit's STDIN is:

    5 |u 1 |i 1
    3 |u 1 |i 2
    4 |u 1 |i 3
    3 |u 1 |i 4
    3 |u 1 |i 5
    5 |u 1 |i 6
    4 |u 1 |i 7
    1 |u 1 |i 8
    5 |u 1 |i 9
    3 |u 1 |i 10
    ...

Testing the model on held-out data results in an average loss of ~0.89 (RMSE of ~0.94):

    awk -F"\t" '{printf "%d |u %d |i %d\n", $3,$1,$2}' < ua.test | \
      ../build/vowpalwabbit/vw /dev/stdin -i movielens.reg -t

Results may vary slightly due to random initialization of the weight vector in the training phase.

# Readable model

The `library/gd_mf_weights.cc` code dumps a readable version of the matrix factorization model to disk. To build it, run `make library_example_build`. Usage is similar to recommend.cc, where you provide a vw parameter string to load up a model and give examples as input, best explained by an example.

For example, extract weights for user 42 and item 7 under a (randomly initialized) rank 10 model:

    echo '|u 42 |i 7' | ./gd_mf_weights -I /dev/stdin --vwparams '-q ui --rank 10'

Presumably you have a model to load. To do this, include a `-i` argument in the vwparams with the name of the model file. For instance, this will show the weights for the same user and item for the trained Movielens model from above:

    echo '|u 42 |i 7' | ./gd_mf_weights -I /dev/stdin --vwparams '-q ui --rank 10 -i movielens.reg'

Five files will be written out:

    constant: a float for the global constant
    outdir/left_ns.linear: featureweight
    outdir/left_ns.quadratic: featurerank1 weightrank2 weight...rankK weight
    outdir/right_ns.linear: featureweight
    outdir/right_ns.quadratic: featurerank1 weightrank2 weight...rankK weight

In the example above, this results in `constant`, `u.linear`, `u.quadratic`, `i.linear`, and `i.quadratic files`.

# Notes on factorization with multiple features and/or namespaces

The matrix factorization code allows factorization over multiple namespaces. You may have multiple features in the same namespace as well as separate namespaces. Both of the examples below differ only syntactically and should provide same results up to differences due to random initialization which is enforced by matrix factorization.

## Multiple namespaces

Lets take an example ``multiple-namespaces.vw``:
```
1 |user 1 |item a |producer P
```

```
$ vw -t -d multiple-namespaces.vw --audit --rank 1 -q ui -q up --quiet | grep "^\t"| tr '\t' "\n" 

u^1:60292(60292):1:0.00678545
i^a:254788(254788):1:0.0143628
p^X:300696(300696):1:0.0243403
464240:1:0.0723715
u1^1:60293(60293):1:0.0619221:i1^a:254789(254789):1:0.0403681:0.00249968
u1^1:60293(60293):1:0.0619221:p1^X:300697(300697):1:0.0563977:0.00349226
```

As you can see above the latent features ``u1^1`` etc are shared across the two ``-q`` factorizations.

## Multiple features in a namespace

Alternatively you can represent your data as in ``multiple-features.vw``:
```
1 |user 1 |item a P
```

```
$ vw -t -d multiple-features.vw --audit --rank 1 -q ui --quiet | grep "^\t"| tr '\t' "\n" 

u^1:60292(60292):1:0.00678545
i^a:254788(254788):1:0.0143628
i^X:748324(748324):1:0.0125303
464240:1:0.0723715
u1^1:60293(60293):1:0.0619221:i1^a:254789(254789):1:0.0403681:0.00249968
u1^1:60293(60293):1:0.0619221:i1^X:748325(748325):1:0.00410265:0.000254045
```

We get three linear features as usual, the numeric feature is Constant and then latent feature interactions. Number of latent features will vary as you vary ``--rank``.