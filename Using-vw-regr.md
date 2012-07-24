The `vw-regr` script is included in the main Vowpal Wabbit distribution, under the `/utl` directory. The script is particularly useful for processing and manipulating datasets with examples having continuously-valued labels. This is the traditional regression framework, exemplified in the batch setting by the closed-form formulas for ordinary least squares (OLS) and ridge regression.

The script is written in Ruby but has no external dependencies beyond the standard installation. More specifically, Ruby gems or the Rails web framework are not required.

`vw-regr` expects to read a stream of Vowpal Wabbit examples from `STDIN`, and expects to write its results (usually modified examples) to `STDOUT`. Feedback and logging is written to `STDERR`. Therefore a typical call to `vw-regr` would like the following:

    $ cat house_dataset |./vw-regr --min_ex_val=-0.01 >selected_house_dataset

This example will filter (remove) any example from the `house_dataset` that has a numeric label of less than -0.01 or -1%. Note that we do not redirect STDERR with the `2>` bash shell syntax, so feedback appears on our console.

Here are the other basic `vw-regr` functions, which can be combined on the same command-line:

***
    --min_ex_val=X
Output just those examples with a label greater than or equal to X. This helps to filter outlier examples.

    --max_ex_val=X
Output just those examples with a label less than or equal to X. This also helps to filter outlier examples.

    --min_num_feats=X
Output just those example with at least X features across all namespaces. This helps to filter examples that are too sparse.

    --max_num_feats=X
Output just those example with no more than X features across all namespaces.

    --pos_ex_val_imp=X
Output the input examples but setting the importance of positively-signed examples to X. This helps to emphasize positive examples during training, especially useful for imbalanced datasets (over-sampling).

    --neg_ex_val_imp=X
Output the input examples but setting the important of negatively-signed examples to X. This helps to emphasize negative examples during training, especially useful for imbalanced datasets (over-sampling).

    --ex_val_desc
Output a description of the input examples, including the number of examples with positive sign, the number with negative sign and the mean & standard deviation of the example labels.

    --vals
Output just the example labels one per line, without importance, tags and features. This is good for importing a column of &ldquo;correct&rdquo; example labels into a column in a spreadsheet.