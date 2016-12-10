Latent Dirichlet Allocation (also called LDA, see http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) is

> "a generative model that allows sets of observations to be explained by unobserved groups that explain why some parts of the data are similar. For example, if observations are words collected into documents, it posits that each document is a mixture of a small number of topics and that each word's creation is attributable to one of the document's topics."

Matt Hoffman published in 2010 a way to perform LDA "online", moving towards a solution in small batches.  See http://www.cs.princeton.edu/~mdhoffma/.  He made code available in Python, and also wrote it into Vowpal.

This video tutorial is useful, but refers to the 5.0 version: http://videolectures.net/nipsworkshops2010_langford_vow/

This tutorial is similar, but with more up to date command-line arguments: https://github.com/JohnLangford/vowpal_wabbit/wiki/lda.pdf

## Friendly wrapper utility to `vw --lda`

The `utl` directory now has a python [utility called `vw-lda`](https://github.com/JohnLangford/vowpal_wabbit/blob/master/utl/vw-lda) which makes interacting with `vw` LDA mode much easier. `utl/vw-lda` does all the necessary pre-processing to convert documents to `vw --lda` format, runs `vw --lda` with good default parameters (which you may optionally override from the command-line) and finally post-processes the results to print all topics in human-readable format and in order of importance as its output. Credit: Chetan Ganjihal.


## Data formats

### Output Topics format

VW can output the topic model in a human-readable textual form, if the `--readable_model` command line parameter is specified while fitting the model. The output format is the following:

* the file starts with a preamble (currently 10 lines) that describes VW version info and model fitting parameters
* each line corresponds to a model dictionary word and is prefixed with word ID. The number of lines is dictated by the choice of feature table size (`-b`)
* columns 2-n represent the per-word topic distributions. The number of topics is specified using the `--lda` parameter.

## Note on LDA Audits

The audit output from VW for LDA may include interaction terms between (i.e. "feat1^feat2:[hash]\t[topic1-weight]\t[topic2-weight]..."), which make it difficult to attribute a single feature to a single topic & feature weight. In this case, you may want to include a dummy namespace after the bar in the training data.
