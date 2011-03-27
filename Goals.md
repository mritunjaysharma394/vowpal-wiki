The general goal of the VW project is to create a very fast, efficient, and capable learning algorithm.  

This is a list of improvements that we want to make to the code.  Any help implementing them is of course welcome.  

<h4>Speed/IO optimizations</h4>:
<ol>
<li>Change the io_buf structure to run in it's own thread.  Currently, reading bits into program space operates synchronously with parsing which implies that delays in the return of read() delay parsing.  This should speedup all input forms (daemon, stdin, file)</li>
<li>Change the text parser to work in a read-once fashion.  Currently, input strings are read multiple times. </li>
<li>Change multisource to use epoll_wait() instead of select().  The amount of speedup is unclear, but it's the right thing to do.</li>
<li>Consider switching the IO subsystem to use <a href="http://code.google.com/apis/protocolbuffers/">protocol buffers</a> which appears mature, and which covers most IO optimizations in the present system.  This will reduce code complexity.</li>
<li>Right now, VW can read from a TCP port, but it shuts down when it's data source shuts down.  We should instead make it persistent.</li>
<li>There should be a way to use the algorithm as a library.</li>
</ol>

<h4>Cluster parallelism improvements</h4>:
<ol>
<li>Semantic multicore.  Right now, multicore learning works quite well (so long as -q is enabled), but the use of multicore learning is limited because the partitioning of the feature space alters the definition of the weights.  We need to make this partitioning work with a single learning thread.</li>
<li>Internal flag passing.  Currently, lots of programs must be started on lots of different machines.  Instead, you should start VW once on the source machine and have it launch other VW process as necessary as well as passing necessary flags (think of rsync).  This is a huge improvement in usability.</li>
<li>Scale-up.  We have a working system for small numbers of nodes, but scaling up to a kilonode-size cluster would be very cool.</li>
</ol>

Algorithmic improvements:
<ol>
<li>Alternate learning algorithms.  The next level up in complexity is confidence weighted updates or matrix factorization style algorithms.  Beyond that, essentially anything trainable in an out-of-core fashion is doable.</li>
<li>Learning reductions.  Previously, we've used VW as a library to implement learning reductions against, but adding a layer of abstraction in the system allowing reductions to directly operate should be doable, and probably desirable.  Especially in a cluster parallel environment, directly supporting learning reductions appears superior to a library implementation.</li>
</ol>
