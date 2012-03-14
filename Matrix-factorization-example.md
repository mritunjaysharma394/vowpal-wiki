The `--rank` option switches VW to matrix factorization mode for interaction terms (as given by `-q`). The argument to `--rank` specifies the rank of the interaction matrix (the number of latent factors) in the model. Simple stochastic gradient descent is performed to update linear and quadratic terms. In contrast to traditional matrix factorization, VW's hashing technique constrains the memory footprint of the learned model, enabling the system to scale to large data sets with many examples and features.

In this mode interaction terms are approximated by a low-rank matrix, which has shown good performance in [model-based collaborative filtering](http://en.wikipedia.org/wiki/Collaborative_filtering#Model-Based) for recommendation systems. See [Matrix Factorization Techniques for Recommender Systems](http://research.yahoo.com/files/ieeecomputer.pdf) by Koren, Bell & Volinksy for an introduction and overview on the topic.

For example, consider the problem of predicting movie ratings for the [Movielens dataset](http://www.grouplens.org/node/73) of 100K ratings: given a user and a movie (which has not yet been rated by that user), predict the rating that the user will assign the movie.

The ratings can be obtained as follows:

    wget http://www.grouplens.org/system/files/ml-100k.zip
    unzip ml-100k.zip
    cd ml-100k

The data consist of `(user, item, rating, date)` events, where ratings are given on an (integer) scale of 1 to 5. Using awk to reformat the data to a VW-friendly format, we can learn a model with a constant term (representing a global average), linear terms (representing per-user and per-item rating biases) and a rank-10 approximation to the interaction terms (representing user-item interactions) as follows:

    awk -F"\t" '{printf "%d |u %d |i %d\n", $3,$1,$2}' < ua.base | \
      vw -b 18 -q ui --rank 10 --l2 0.001 \
      --learning_rate 0.025 --passes 20 --decay_learning_rate 0.97 --power_t 0 \
      -f movielens.reg --cache_file movielens.cache

Note that the combination of `-b 18` and `--rank 10` results in a weight vector of (1+2*10)*2^18 elements. The `--l2` option is the L2 regularization argument to avoid overfitting.

Testing the model on held-out data results in an average loss of ~0.89 (RMSE of ~0.94):

    awk -F"\t" '{printf "%d |u %d |i %d\n", $3,$1,$2}' < ua.test | \
      vw -q ui --rank 10 -i movielens.reg -t

Results may vary slightly due to random initialization of the weight vector in the training phase.