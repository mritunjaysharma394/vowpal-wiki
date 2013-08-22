Latent Dirichlet Allocation (also called LDA, see http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) is

> "a generative model that allows sets of observations to be explained by unobserved groups that explain why some parts of the data are similar. For example, if observations are words collected into documents, it posits that each document is a mixture of a small number of topics and that each word's creation is attributable to one of the document's topics."

Matt Hoffman published in 2010 a way to perform LDA "online", moving towards a solution in small batches.  See http://www.cs.princeton.edu/~mdhoffma/.  He made code available in Python, and also wrote it into Vowpal.

This video tutorial is useful, but refers to the 5.0 version: http://videolectures.net/nipsworkshops2010_langford_vow/

This tutorial is similar, but with more up to date command-line arguments: https://github.com/JohnLangford/vowpal_wabbit/wiki/lda.pdf

### Note on LDA Audits

The audit output from VW for LDA includes interaction terms (i.e. "feat1^feat2:[hash]\t[topic1-weight]\t[topic2-weight]..."), which make it difficult to attribute a single feature to a single topic & feature weight. In this case, Matt Hoffman recommends doing your own hashing before calling VW, and then interpreting the readable model ("--readable_model") output instead of the audit.
