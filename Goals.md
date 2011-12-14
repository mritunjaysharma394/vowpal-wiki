The general goal of the VW project is to create a very fast, efficient, and capable learning algorithm.  

This is a list of improvements that we want to make to the code.  Any help implementing them is of course welcome.  

<h4>Speed/IO optimizations</h4>
<ol>
<li>Change the io_buf structure to run in it's own thread.  Currently, reading bits into program space operates synchronously with parsing which implies that delays in the return of read() delay parsing.  This should speedup all input forms (daemon, stdin, file)</li>
<li>Change the text parser to work in a read-once fashion.  Currently, input strings are read multiple times. </li>
<li>There should be a way to use the algorithms as a library.</li>
</ol>

<h4>Algorithmic improvements</h4>
<ol>
<li>Alternate learning algorithms.  We have basic matrix factorization, which needs to be developed further.  We also want to push into more complex nonconvex algorithms.</li>
<li>Learning reductions.  Previously, we've used VW as a library to implement learning reductions against, but adding a layer of abstraction in the system allowing reductions to directly operate should be doable, and desirable.  Especially in a cluster parallel environment, directly supporting learning reductions appears superior to a library implementation.</li>
</ol>
