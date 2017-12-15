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


### Programming principles

This is an attempt to capture (from John Langford) the principles of programming in Vowpal Wabbit that should be kept in mind by anyone contributing to the source code. 

1. All memory is initialized to 0 (unless it has a constructor).
2. Memory allocation is avoided by reusing allocated memory where possible. 
3. Floats are preferred over doubles.   Doubles are only used for accumulators.
4. Templates are used to eliminate duplicate code and in some places to remove branches from inner loops.
5. Pass by reference is the default, except for objects of pointer size.  (Some older code is pass-by-pointer.)
6. All learning reductions are confined to a single file with a single entry point. 
7. Learning reductions transform an example from one problem type to another.  (A problem type is defined by (label, prediction, features).
8. The execution of the machine is explicitly traceable:
* Memory allocation is explicit (... except for portions of the code by some people who love objects)
* Memory destruction is explicit.
* Function pointer interfaces are explicit.
9. All I/O of core objects is by reversible functions.
10. All examples are handled by the same stack of reductions.  
11. Examples are passed by function call.  
12. It’s not working until there are no warnings and valgrind says it’s clean.
