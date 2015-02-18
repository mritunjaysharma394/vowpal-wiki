<h3>Vowpal Wabbit</h3>

The <a href="http://hunch.net/~vw">Vowpal Wabbit</a> (VW) project is a fast out-of-core learning system sponsored by <a href="http://research.microsoft.com/en-us/">Microsoft Research</a> and (previously) <a href="http://research.yahoo.com/node/1914">Yahoo! Research</a>. Support is available through the <a href="https://groups.yahoo.com/neo/groups/vowpal_wabbit/info">mailing list</a>.

There are two ways to have a fast learning algorithm: (a) start with a slow algorithm and speed it up, or (b) build an intrinsically fast learning algorithm. This project is about approach (b), and it's reached a state where it may be useful to others as a platform for research and experimentation.

There are several optimization algorithms available with the baseline being sparse gradient descent (GD) on a loss function (several are available), The code should be easily usable. Its only external dependence is on the <a href="http://www.boost.org/">boost library</a>, which is often installed by default. 

<ul>
<li>[[Download]]</li>
<li>[[Tutorial]]</li>
<li>[[Command line arguments]]</li>
<li>[[Algorithm details]] (e.g., [[input format]], [[loss functions]])</li>
<li>[[Examples]]</li>
<li>[[Goals]]</li>
<li>[[Feature Hashing and Extraction]]</li>
<li><a href="http://www.umiacs.umd.edu/~hal/tmp/multiclassVW.html">Multiclass Classification</a></li>
<li><a href="http://nbviewer.ipython.org/github/hal3/vowpal_wabbit/blob/master/python/Learning_to_Search.ipynb">Python interface for learning to search</a></li>

</ul>

<h2>Features</h2>
There are several features that (in combination) can be powerful.
<ol>
<li><strong>Input Format</strong>.  The <a href="https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format">input format</a> for the learning algorithm is substantially more flexible than might be expected.  Examples can have features consisting of free form text, which is interpreted in a bag-of-words way.  There can even be multiple sets of free form text in different namespaces.</li>
<li><strong>Speed</strong>.  The learning algorithm is pretty fast---similar to the few other online algorithm implementations out there.  As one datapoint, it can be effectively applied on learning problems with a sparse terafeature (i.e. 10<sup>12</sup> sparse features).  As another example, it's about a factor of 3 faster than <a href="http://leon.bottou.org/">Leon Bottou</a>'s <a href="http://leon.bottou.org/projects/sgd">svmsgd</a> on the [[RCV1 example]] in wall clock execution time.</li>
<li><strong>Scalability</strong>.  This is not the same as fast.  Instead, the important characterictic here is that the memory footprint of the program is bounded independent of data.  This means the training set is not loaded into main memory before learning starts.  In addition, the size of the set of features is bounded independent of the amount of training data using the <a href="http://hunch.net/~jl/projects/hash_reps/index.html">hashing trick</a>.</li>
<li><strong>Feature Pairing</strong>.  Subsets of features can be internally paired so that the algorithm is linear in the cross-product of the subsets.  This is useful for ranking problems.  <a href="http://www.idiap.ch/~grangier/">David Grangier</a> seems to have a similar trick in the <a href="http://www.idiap.ch/pamir/">PAMIR code</a>.  The alternative of explicitly expanding the features before feeding them into the learning algorithm can be both computation and space intensive, depending on how it's handled.</li>
</ol>

Many people have contributed to the project at this point.
<a href="http://hunch.net/~jl">John Langford</a>, <a href="http://www.cs.berkeley.edu/~alekh/">Alekh Agarwal</a>, <a href="http://www.cs.cmu.edu/~mdudik/">Miroslav Dudik</a>, <a href="http://cseweb.ucsd.edu/~djhsu/">Daniel Hsu</a>, <a href="http://www.cs.cornell.edu/~nk/">Nikos Karampatziakis</a>, <a href="http://research.yahoo.com/Olivier_Chapelle">Olivier Chapelle</a>, <a href="http://www.machinedlearnings.com/">Paul Mineiro</a>, <a href=http://www.cs.princeton.edu/~mdhoffma>Matt Hoffman</a>, <a href="http://jakehofman.com">Jake Hofman</a>, <a href="http://labs.yahoo.com/Sudarshan_Lamkhede">Sudarshan Lamkhede</a>, Shubham Chopra, Ariel Faigon, <a href="http://www.research.rutgers.edu/~lihong/">Lihong Li</a>, Gordon Rios, and <a href="http://paul.rutgers.edu/~strehl/">Alex Strehl</a> have all worked on VW.  Many others have contributed via feature requests, bug reports, or bug patches.

<h2>Research</h2>
VW is also a vehicle for advanced research.  The <a href="http://hunch.net/?p=309">first public version</a> containing hashing, caching, and true online learning was released in 2007.  Since then, many different algorithms and results have influenced its design, including:

1. <a href="http://www.cs.berkeley.edu/~alekh/">Alekh Agarwal</a>, <a href="http://olivier.chapelle.cc/">Olivier Chapelle</a>, <a href="http://www.cs.cmu.edu/~mdudik/">Miroslav Dudik</a>, <a href="http://hunch.net/~jl">John Langford</a>, <a href="http://arxiv.org/abs/1110.4198">A Reliable Effective Terascale Linear Learning System</a>, 2011.
2. <a href="http://www.cs.princeton.edu/~mdhoffma/">M. Hoffman</a>, <a href="http://www.cs.princeton.edu/~blei/">D. Blei</a>, <a href="http://www.di.ens.fr/~fbach/">F. Bach</a>, <a href="http://www.cs.princeton.edu/~blei/papers/HoffmanBleiBach2010b.pdf">Online Learning for Latent Dirichlet Allocation</a>, in Neural Information Processing Systems (NIPS) 2010.
3. <a href="http://hunch.net/~beygel">Alina Beygelzimer</a>, <a href="http://cseweb.ucsd.edu/~djhsu/">Daniel Hsu</a>, <a href="http://hunch.net/~jl">John Langford</a>, and <a href="http://stat.rutgers.edu/home/tzhang/">Tong Zhang</a> <a href="http://arxiv.org/abs/1006.2588">Agnostic Active Learning Without Constraints</a> NIPS 2010.
4. <a href="http://www.cs.berkeley.edu/~jduchi/">John Duchi</a>, <a href="http://ie.technion.ac.il/~ehazan/">Elad Hazan</a>, and <a href="http://www.magicbroom.info/About.html">Yoram Singer</a>, <a href="http://www.cs.berkeley.edu/~jduchi/projects/DuchiHaSi10.html">Adaptive Subgradient Methods for Online Learning and Stochastic Optimization</a>, JMLR 2011 & COLT 2010.
5. <a href="http://www.cs.cmu.edu/~mcmahan/">H. Brendan McMahan</a>, <a href="http://www.cs.cmu.edu/~matts/">Matthew Streeter</a>, <a href="http://arxiv.org/abs/1002.4908">Adaptive Bound Optimization for Online Convex Optimization</a>, COLT 2010.
6. <a href="http://www.cs.cornell.edu/~nk/">Nikos Karampatziakis</a> and <a href="http://hunch.net/~jl">John Langford</a>, <a href="http://arxiv.org/abs/1011.1576">Importance Weight Aware Gradient Updates</a> UAI 2010.
7. <a href="http://www.cse.wustl.edu/~kilian/">Kilian Weinberger</a>, <a href="http://research.yahoo.com/Anirban_Dasgupta/">Anirban Dasgupta</a>, <a href="http://hunch.net/~jl">John Langford</a>, <a href="http://alex.smola.org/">Alex Smola</a>, <a href="http://www.linkedin.com/in/joshattenberg">Josh Attenberg</a>, <a href="http://arxiv.org/pdf/0902.2206">Feature Hashing for Large Scale Multitask Learning</a>, ICML 2009.
8. <a href="http://users.cecs.anu.edu.au/~qshi/">Qinfeng Shi</a>, <a href="http://users.cecs.anu.edu.au/~jpetterson/">James Petterson</a>, <a href="http://www2.mta.ac.il/~gideon/">Gideon Dror</a>, <a href="http://hunch.net/~jl">John Langford</a>, <a href="http://alex.smola.org/">Alex Smola</a>, and <a href="http://www.stat.purdue.edu/~vishy/">SVN Vishwanathan</a>, <a href="http://hunch.net/~jl/projects/hash_reps/hash_kernels/hashkernel.pdf">Hash Kernels for Structured Data</a>, AISTAT 2009.
9. <a href="http://hunch.net/~jl">John Langford</a>, <a href="http://www.research.rutgers.edu/~lihong/">Lihong Li</a>, and <a href="http://stat.rutgers.edu/home/tzhang/">Tong Zhang</a>, <a href="http://hunch.net/~jl/projects/interactive/sparse_online/paper_sparseonline.pdf">Sparse Online Learning via Truncated Gradient</a>, NIPS 2008.
10. <a href="http://leon.bottou.org/">Leon Bottou</a>, <a href="http://leon.bottou.org/projects/sgd">Stochastic Gradient Descent</a>, 2007.
11. <a href="http://www.cs.cmu.edu/~avrim/">Avrim Blum</a>, <a href="http://www.cs.cmu.edu/~akalai/">Adam Kalai</a>, and <a href="http://hunch.net/~jl">John Langford</a> <a href="http://hunch.net/~jl/projects/prediction_bounds/progressive_validation/coltfinal.pdf">Beating the Holdout: Bounds for KFold and Progressive Cross-Validation</a>. COLT99 pages 203-208.
12. <a href="http://www.ece.northwestern.edu/faculty/Nocedal_Jorge.html">Nocedal, J.</a> (1980). "Updating Quasi-Newton Matrices with Limited Storage". Mathematics of Computation 35: 773–782.