VW has a --warm_cb option that simulates [warm-starting contextual bandits learning](https://arxiv.org/pdf/1901.00301.pdf). In this setting, the learner is given a set of warm-start supervised learning examples to help with contextual bandit learning. With the help of these additional warm-start examples, the learner is able to achieve a smaller cost in the interaction stage.

The learning process consists of two stages:

1. Warm-start. The learner receives warm-start examples and updates its model accordingly.
2. Interaction. The learner starts with the warm-started model, perform online contextual bandit learning, alternating between prediction and update. 

The performance of the learner is measured by its cost incurred in the interaction stage.

VW takes into input option --warm_start x --interaction y, where x specifies the number of warm start examples, and y is the length of the interaction stage. 

# 1 A simple example

Suppose we have [text_highnoise_m.vw](https://raw.githubusercontent.com/zcc1307/vw_datasets/master/datasets/text_highnoise_m.vw), a dataset of 10-class multiclass examples in VW format. We can run:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update --interaction_update -d text_highnoise_m.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0        4        4      101
    0.000000 0.000000           12            2.0        6        6      101
    0.000000 0.000000           14            4.0        6        6      101
    0.125000 0.250000           18            8.0        8        5      101
    0.125000 0.125000           26           16.0        8        8      101
    0.125000 0.125000           42           32.0       10       10      101
    0.093750 0.062500           74           64.0        9        6      101
    0.117188 0.140625          138          128.0        5        5      101
    0.132812 0.148438          266          256.0        9        3      101
    0.134766 0.136719          522          512.0        6        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.132000
    total feature number = 1010000
    average variance estimate = 10.795802
    theoretical average variance = 200.000000
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000


Note that the VW output has the same doubling schedule; however, we only count the example weight, the average loss, and the loss since last checkpoint in the interaction stage. (The "example counter" starts from 10 though - this is because we processed the first 10 warm start examples before the interaction stage.)

# 2 Baseline 1: using only interaction examples (Bandit-Only)

We can also run the above command without the "--warm_start_update" option, which essentially skips the warm start examples and perform contextual bandit learning directly:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --interaction_update -d text_highnoise_m.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    1.000000 1.000000           11            1.0        4        5      101
    1.000000 1.000000           12            2.0        6        9      101
    1.000000 1.000000           14            4.0        6        7      101
    1.000000 1.000000           18            8.0        8        6      101
    0.875000 0.750000           26           16.0        8        1      101
    0.937500 1.000000           42           32.0       10        9      101
    0.859375 0.781250           74           64.0        9        6      101
    0.687500 0.515625          138          128.0        5        5      101
    0.453125 0.218750          266          256.0        9        9      101
    0.283203 0.113281          522          512.0        6        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.166000
    total feature number = 1010000
    average variance estimate = 22.381275
    theoretical average variance = 200.000000
    last lambda chosen = 1.000000 among lambdas ranging from 1.000000 to 1.000000

# 3 Baseline 2: using only warm-start examples (Sup-Only)

Another extreme is to run contextual bandit learning by training a model purely based on the warm start examples and, ignore the data collected in the interaction stage. In this case, we turn off exploration to minimize the exploration overhead.

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update -d text_highnoise_m.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0        4        4      101
    0.000000 0.000000           12            2.0        6        6      101
    0.000000 0.000000           14            4.0        6        6      101
    0.000000 0.000000           18            8.0        8        8      101
    0.062500 0.125000           26           16.0        8        8      101
    0.093750 0.125000           42           32.0       10       10      101
    0.109375 0.125000           74           64.0        9        3      101
    0.148438 0.187500          138          128.0        5        5      101
    0.156250 0.164062          266          256.0        9        3      101
    0.150391 0.144531          522          512.0        6        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.148000
    total feature number = 1010000
    average variance estimate = 1.047119
    theoretical average variance = 200.000000
    last lambda chosen = 0.000000 among lambdas ranging from 0.000000 to 0.000000

# 4 Label corruption on the warm-starting examples

Sometimes, we would like to simulate the setting where the label distribution of the warm-start examples does not perfectly match those of the examples in the interaction stage. VW supports this by allowing to specify one of three modes of corruption, and the corruption probability (specified in --corrupt_type_warm_start and --corrupt_prob_warm_start). The three modes or corruptions are:
1. replace a label with one chosen uniformly-at-random; 
2. replace label i with its next label ((i+1) mod K, where K is the total number of classes) 
3. replace a label with a "overwriting" label l. 

For example, we can add type 2 corruption with probability 0.5 on the first 10 supervised learning examples:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update --interaction_update --corrupt_type_warm_start 2 --corrupt_prob_warm_start 0.5 -d text_highnoise_m.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0        4        4      101
    0.500000 1.000000           12            2.0        6        1      101
    0.500000 0.500000           14            4.0        6        7      101
    0.500000 0.500000           18            8.0        8        5      101
    0.562500 0.625000           26           16.0        8        1      101
    0.593750 0.625000           42           32.0       10        1      101
    0.593750 0.593750           74           64.0        9        2      101
    0.632812 0.671875          138          128.0        5        5      101
    0.593750 0.554688          266          256.0        9        8      101
    0.529297 0.464844          522          512.0        6       10      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.436000
    total feature number = 1010000
    average variance estimate = 15.968479
    theoretical average variance = 200.000000
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000

# 5 Using a larger set of weighted combination values

By default, the warm-start contextual bandit learner place equal weights on every warm-start examples and every interaction examples. It is often a good idea to use a large set of weighted combination values and perform selection on top of them, to find out the right balance between warm-start examples and the bandit examples in the interaction stage. This can be specified with the --lambda_scheme parameter and --choices_lambda_parameter, as shown in the example below.

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update --interaction_update --corrupt_type_warm_start 2 --corrupt_prob_warm_start 0.5 --lambda_scheme 2 --choices_lambda 2 -d text_hig
    hnoise_m.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0        4        4      101
    0.500000 1.000000           12            2.0        6        1      101
    0.750000 1.000000           14            4.0        6        7      101
    0.500000 0.250000           18            8.0        8        8      101
    0.687500 0.875000           26           16.0        8        4      101
    0.687500 0.687500           42           32.0       10        9      101
    0.656250 0.625000           74           64.0        9        3      101
    0.570312 0.484375          138          128.0        5        5      101
    0.433594 0.296875          266          256.0        9        1      101
    0.304688 0.175781          522          512.0        6        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.177000
    total feature number = 1010000
    average variance estimate = 27.510756
    theoretical average variance = 200.000000
    last lambda chosen = 1.000000 among lambdas ranging from 0.000000 to 1.000000

# 6 Allowing cost-sensitive examples as input

Warm-cb also supports the input examples being of cost-sensitive form, i.e. each example's label part is a cost vector. To accept this input format, VW needs to take an additional option --warm_cb_cs. Here we have a dataset [text_highnoise.vw](https://raw.githubusercontent.com/zcc1307/vw_datasets/master/datasets/text_highnoise.vw), which is identical to text_highnoise_m.vw, except that each example's label is in the cost vector format. We get exactly the same result as running using text_highnoise_m.vw as input without --warm_cb_cs option:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update --interaction_update --warm_cb_cs -d text_highnoise.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0    known        4      101
    0.000000 0.000000           12            2.0    known        6      101
    0.000000 0.000000           14            4.0    known        6      101
    0.125000 0.250000           18            8.0    known        5      101
    0.125000 0.125000           26           16.0    known        8      101
    0.125000 0.125000           42           32.0    known       10      101
    0.093750 0.062500           74           64.0    known        6      101
    0.117188 0.140625          138          128.0    known        5      101
    0.132812 0.148438          266          256.0    known        3      101
    0.134766 0.136719          522          512.0    known        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.132000
    total feature number = 1010000
    average variance estimate = 10.795802
    theoretical average variance = 200.000000
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000


# 7 Baseline 3: Contextual bandit simulation (Sim-Bandit)

We also include a baseline approach, named Sim-Bandit in the [warm contextual bandits paper](https://arxiv.org/pdf/1901.00301.pdf). In the warm-start stage, it performs simulation of contextual bandit learning and produces a model; then, in the interaction stage, it continues contextual bandit learning, with the model initialized as the one at the end of the warm-start stage. This under-utilizes the warm-start examples, as the algorithm only uses part of the label for every warm-start example.

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 1000 --warm_start_update --interaction_update --warm_cb_cs -d text_highnoise.vw
    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    0.000000 0.000000           11            1.0    known        4      101
    0.000000 0.000000           12            2.0    known        6      101
    0.000000 0.000000           14            4.0    known        6      101
    0.125000 0.250000           18            8.0    known        5      101
    0.125000 0.125000           26           16.0    known        8      101
    0.125000 0.125000           42           32.0    known       10      101
    0.093750 0.062500           74           64.0    known        6      101
    0.117188 0.140625          138          128.0    known        5      101
    0.132812 0.148438          266          256.0    known        3      101
    0.134766 0.136719          522          512.0    known        6      101

    finished run
    number of examples = 10000
    weighted example sum = 1000.000000
    weighted label sum = 0.000000
    average loss = 0.132000
    total feature number = 1010000
    average variance estimate = 10.795802
    theoretical average variance = 200.000000
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000
