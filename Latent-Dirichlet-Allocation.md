Latent Dirichlet Allocation (also called LDA, see http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) is

> "a generative model that allows sets of observations to be explained by unobserved groups that explain why some parts of the data are similar. For example, if observations are words collected into documents, it posits that each document is a mixture of a small number of topics and that each word's creation is attributable to one of the document's topics."

Matt Hoffman published in 2010 a way to perform LDA "online", moving towards a solution in small batches.  See http://www.cs.princeton.edu/~mdhoffma/.  He made code available in Python, and also wrote it into Vowpal.

This video tutorial is useful, but refers to the 5.0 version: http://videolectures.net/nipsworkshops2010_langford_vow/

This tutorial is similar, but with more up to date command-line arguments: https://github.com/JohnLangford/vowpal_wabbit/wiki/lda.pdf

Note from Dan, June 2011: I do not know how to get 5.1 to work, it appears to be missing the --readable_topics argument.  So, I use head.