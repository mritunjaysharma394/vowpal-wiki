VW has --warm_cb option which simulates [warm-starting contextual bandits learning](https://arxiv.org/pdf/1901.00301.pdf). In this setting, the learner is given a set of warm-start supervised learning examples to help with contextual bandit learning. With the help of these additional warm-start examples, the learner is able to achieve a smaller cost in the interaction stage.

The learning process consists of two stages:

1. Warm-start. The learner receives warm-start examples and updates its model accordingly.
2. Interaction. The learner starts with the warm-started model, perform online contextual bandit learning, alternating between prediction and update. 

The performance of the learner is measured by its cost incurred in the interaction stage.

VW takes into input option --warm_start x --interaction y, where x specifies the number of warm start examples, and y is the length of the interaction stage. 

### A simple example

Suppose we have text_highnoise_m.vw, a dataset of 10-class multiclass examples in VW format. We can run:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.0 --warm_start 10 --interaction 5000 --warm_start_update --interaction_update -d text_highnoise_m.vw

and get the following output:

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
    0.078125 0.062500           74           64.0        9        3      101
    0.078125 0.078125          138          128.0        5        5      101
    0.093750 0.109375          266          256.0        9        3      101
    0.093750 0.093750          522          512.0        6        6      101
    0.090820 0.087891         1034         1024.0        5        5      101
    0.100098 0.109375         2058         2048.0        2        2      101
    0.094727 0.089355         4106         4096.0        2        2      101

    finished run
    number of examples = 10000
    weighted example sum = 5000.000000
    weighted label sum = 0.000000
    average loss = 0.096400
    total feature number = 1010000
    average variance estimate = inf
    theoretical average variance = inf
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000

Note that the VW output has the same doubling schedule; however, we only count the example weight, the average loss, and the loss since last checkpoint in the interaction stage. (The "example counter" starts from 10 though - this is because we processed the first 10 warm start examples before the interaction stage.)

We can also run the above command without the "--warm_start_update" option, which essentially skips the warm start examples and perform contextual bandit learning directly:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 30 --interaction 5000 --interaction_update -d text_highnoise_m.vw

    Num weight bits = 18
    learning rate = 0.5
    initial_t = 0
    power_t = 0.5
    using no cache
    Reading datafile = text_highnoise_m.vw
    num sources = 1
    average  since         example        example  current  current  current
    loss     last          counter         weight    label  predict features
    1.000000 1.000000           31            1.0        8        5      101
    1.000000 1.000000           32            2.0        1        9      101
    1.000000 1.000000           34            4.0        6        7      101
    1.000000 1.000000           38            8.0        9        3      101
    1.000000 1.000000           46           16.0        4        5      101
    0.937500 0.875000           62           32.0        4        4      101
    0.906250 0.875000           94           64.0        6        9      101
    0.789062 0.671875          158          128.0        9        9      101
    0.593750 0.398438          286          256.0       10       10      101
    0.427734 0.261719          542          512.0        2        2      101
    0.283203 0.138672         1054         1024.0       10       10      101
    0.181152 0.079102         2078         2048.0        5        5      101
    0.114014 0.046875         4126         4096.0        5        5      101

    finished run
    number of examples = 10000
    weighted example sum = 5000.000000
    weighted label sum = 0.000000
    average loss = 0.101000
    total feature number = 1010000
    average variance estimate = 8.695751
    theoretical average variance = 200.000000
    last lambda chosen = 1.000000 among lambdas ranging from 1.000000 to 1.000000

Another extreme is to run contextual bandit learning without using the interaction data; in this case, we can simply turn off exploration to minimize the exploration overhead.

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.0 --warm_start 10 --interaction 5000 --warm_start_update -d text_highnoise_m.vw

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
    0.117188 0.125000          138          128.0        5        5      101
    0.121094 0.125000          266          256.0        9        3      101
    0.111328 0.101562          522          512.0        6        6      101
    0.109375 0.107422         1034         1024.0        5        5      101
    0.116211 0.123047         2058         2048.0        2        2      101
    0.110352 0.104492         4106         4096.0        2        2      101

    finished run
    number of examples = 10000
    weighted example sum = 5000.000000
    weighted label sum = 0.000000
    average loss = 0.112000
    total feature number = 1010000
    average variance estimate = 1.000000
    theoretical average variance = inf
    last lambda chosen = 0.000000 among lambdas ranging from 0.000000 to 0.000000

### Label corruption on the warm-starting examples

Sometimes, we would like to simulate the setting where the label distribution of the warm-start examples does not perfectly match those of the examples in the interaction stage. VW supports this by allowing to specify one of three modes of corruption, and the corruption probability (specified in --corrupt_type_warm_start and --corrupt_prob_warm_start). The three modes or corruptions are:
1. replace a label with one chosen uniformly-at-random; 
2. replace label i with its next label ((i+1) mod K, where K is the total number of classes) 
3. replace a label with a "overwriting" label l. 

For example, we can add type 2 corruption with probability 0.5 on the first 10 supervised learning examples:

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 5000 --warm_start_update --interaction_update --corrupt_type_warm_start 2 --corrupt_prob_warm_start 0.5 -d text_highnoise_m.vw

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
    0.500000 0.500000           18            8.0        8        9      101
    0.562500 0.625000           26           16.0        8        4      101
    0.593750 0.625000           42           32.0       10        1      101
    0.593750 0.593750           74           64.0        9        1      101
    0.632812 0.671875          138          128.0        5        5      101
    0.593750 0.554688          266          256.0        9        4      101
    0.628906 0.664062          522          512.0        6        7      101
    0.550781 0.472656         1034         1024.0        5        5      101
    0.435547 0.320312         2058         2048.0        2        2      101
    0.310547 0.185547         4106         4096.0        2        2      101

    finished run
    number of examples = 10000
    weighted example sum = 5000.000000
    weighted label sum = 0.000000
    average loss = 0.262000
    total feature number = 1010000
    average variance estimate = 2.757993
    theoretical average variance = 200.000000
    last lambda chosen = 0.500000 among lambdas ranging from 0.500000 to 0.500000

### Using a larger set of weighted combination values

It is often a good idea to use a large set of weighted combination values and perform selection on top of them, to find out the right balance between warm-start examples and the bandit examples in the interaction stage. This can be specified with the --lambda_scheme parameter and --choices_lambda_parameter, as shown in the example below.

    ./vw --warm_cb 10 --cb_explore_adf --cb_type mtr --epsilon 0.05 --warm_start 10 --interaction 5000 --warm_start_update --interaction_update --corrupt_type_warm_start 2 --corrupt_prob_warm_start 0.5 --lambda_scheme 2 --choices_lambda 2 -d text_highnoise_m.vw 
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
    0.734375 0.781250           74           64.0        9        5      101
    0.726562 0.718750          138          128.0        5        5      101
    0.601562 0.476562          266          256.0        9        9      101
    0.576172 0.550781          522          512.0        6        7      101
    0.381836 0.187500         1034         1024.0        5        5      101
    0.236816 0.091797         2058         2048.0        2        2      101
    0.142090 0.047363         4106         4096.0        2        2      101

    finished run
    number of examples = 10000
    weighted example sum = 5000.000000
    weighted label sum = 0.000000
    average loss = 0.124000
    total feature number = 1010000
    average variance estimate = 13.621336
    theoretical average variance = 200.000000
    last lambda chosen = 1.000000 among lambdas ranging from 0.000000 to 1.000000
