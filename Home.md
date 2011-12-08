<h3>Vowpal Wabbit</h3>

The <a href="http://hunch.net/~vw">Vowpal Wabbit</a> (VW) project is a fast out-of-core learning system sponsored by <a href="http://research.yahoo.com/node/1914">Yahoo! Research</a>. Support is available through the <a href="http://tech.groups.yahoo.com/group/vowpal_wabbit/">mailing list</a>.

There are two ways to have a fast learning algorithm: (a) start with a slow algorithm and speed it up, or (b) build an intrinsically fast learning algorithm. This project is about approach (b), and it's reached a state where it may be useful to others as a platform for research and experimentation.

There are several optimization algorithms available with the baseline being sparse gradient descent (GD) on a loss function (several are available), The code should be easily usable. Its only external dependence is on the <a href="http://www.boost.org/">boost library</a>, which is often installed by default. 

<ul>
<li>[[Download]]</li>
<li>[[Tutorial]]</li>
<li>[[Command line arguments]]</li>
<li>[[Algorithm details]] (e.g., [[input format]], [[loss functions]])</li>
<li>[[Examples]]</li>
<li>[[Goals]]</li>
<li>[[Discussions]]</li>

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
<a href="http://hunch.net/~jl">John Langford</a>, <a href="http://cseweb.ucsd.edu/~djhsu/">Daniel Hsu</a>, <a href="http://www.cs.cornell.edu/~nk/">Nikos Karampatziakis</a>, <a href="http://research.yahoo.com/Olivier_Chapelle">Olivier Chapelle</a>, <a href="http://www.machinedlearnings.com/">Paul Mineiro</a>, Matt Hoffman, Jake Hofman, <a href="http://labs.yahoo.com/Sudarshan_Lamkhede">Sudarshan Lamkhede</a>, Shubham Chopra, Ariel Faigon, <a href="http://www.research.rutgers.edu/~lihong/">Lihong Li</a>, Gordon Rios, and <a href="http://paul.rutgers.edu/~strehl/">Alex Strehl</a> have all worked on VW.  Many others have contributed via feature requests, bug reports, or bug patches.