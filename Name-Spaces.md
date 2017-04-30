vowpal wabbit supports a concept called *feature name spaces* (aka _namespaces_).

Two identically named features in two separate name-spaces are effectively different features.

Name spaces are useful for:

* Separating the data-set into example subsets (think one model including multiple, smaller and separate sub-models)
* Separating _features_ (columns) into subsets
* Ignore/drop a subset of features belonging to a name-space as a group from consideration
* Cross one name-space subset with another, on the fly during runtime (dynamic feature-interaction generation).

Example1: row-based name-spaces: suppose you're tracking revenues for N different stores, based on features like: year opened, median income in local zip-code, Previous-week gross proceeds, day-of-week. etc.  You can use the store name or town as a name-space for all the other features segregating the data-set into N separate sub-models, one for each store.  Obviously, when trying to predict, you should be sending the expected name-space with the example you want to predict.

Example2: column-based name-spaces: you have features about a product (price, category, ...) in one name-space P and features about the buyers of the product (age, income, ...) in another name-space B, by using `--quadratic PB` you can generate the interaction features between the two name-spaces on-the-fly.  

## Input format:

In essence when you add a name-space to an example, all the features following that name-space get mapped to a new feature name.  A name-space is introduced by dropping the space after the `|` char introducing input features.

| Namespace  | Original feature-name   | Effective feature name  | Example syntax (note presence/absence of space after `\|`)  |
|------------|-------------------------|-------------------------|-----------------|
|            | foo                     | foo                     | \| foo          |
| Bar        | foo                     | Bar^foo                 | \|Bar foo       |
| Bar        | baz                     | Bar^baz                 | \|Bar baz       |


## Command-line options related to name-spaces:

A few of the many `vw` command-line options, act on name spaces.

Limitation: options that act on name-spaces only use on the 1st char of the name-space. You cannot (for example) drop only one name-space starting with `a` if you have more than one name-space starting with `a`.

| Option                    | Meaning                                                    |
|---------------------------|------------------------------------------------------------|
| `--keep c`                | Keep a name-space staring with the character `c`           |
| `--ignore c`              | Ignore a name-space starting with the character `c`        |
| `--redefine a:=b`         | redefine namespace starting with `b` as starting with `a`  |
| `--quadratic ab`          | Cross namespaces starting with `a` & `b` on the fly to generate 2-way interacting features  |
| `--cubic abc`             | Cross namespaces starting with `a`, `b`, & `c` on the fly to generate 3-way interacting features  |

The complete [Input file format](Input-format) page covers the full syntax.


## Further reading:

* [vw command line arguments](Command-line-arguments)
* [vw input format spec](Input-format)
* [(Stackoverflow): Difference between name space and feature](http://stackoverflow.com/questions/28586225/in-vowpal-wabbit-what-is-the-difference-between-a-namespace-and-feature)
* [(Crossvalidated): Finding the best features in interaction models](https://stats.stackexchange.com/questions/28877/finding-the-best-features-in-interaction-models/32728)
* [(Stackoverflow): Can't use full-name of name space in --keep](http://stackoverflow.com/questions/30055043/vowpal-wabbit-specifying-the-full-name-of-a-namespace-with-option-keep)
* [(Stackoverflow): What hash function is used by vw?](http://stackoverflow.com/questions/32119754/vowpal-wabbit-what-hash-function-is-used-exactly)
* [(Stackoverflow): How to use the `--keep` and `--ignore` options?](http://stackoverflow.com/questions/24662721/how-does-one-use-the-keep-and-ignore-features-of-vowpal-wabbit)