_This page is primarily targeted at maintainers and contributors_ 

The idea of these guidelines is to form a general consensus about the method used and help progress towards the general goal of the VW project to create a very fast, efficient, and capable learning algorithm

# Guidelines

1. All memory is initialized to 0 (unless it has a constructor).
2. Memory allocation is avoided by reusing allocated memory where possible. 
3. Floats are preferred over doubles.   Doubles are only used for accumulators.
4. Templates are used to eliminate duplicate code and in some places to remove branches from inner loops.
5. Pass by reference is the default, except for objects of pointer size.  (Some older code is pass-by-pointer.)
6. All learning reductions are confined to a single file with a single entry point. 
7. Learning reductions transform an example from one problem type to another.  
    * A problem type is defined by (label, prediction, features)
8. The execution of the machine is explicitly traceable:
    * Memory allocation is explicit (... except for portions of the code by some people who love objects)
    * Memory destruction is explicit.
    * Function pointer interfaces are explicit.
9. All I/O of core objects is by reversible functions.
10. All examples are handled by the same stack of reductions.  
11. Examples are passed by function call.  
12. It’s not working until there are no warnings and Valgrind says it’s clean.
13. Prefer to use fixed size types where possible. Example: `uint32_t`

## Exception Policy
1. If an error state can be handled locally, that should always be what we do.  
2. All memory allocation should go through `memory.h` with errors handled by exception (the default) or crash (conditional compile).  In the future, we may add the ability to specify a memory allocator.
3. No new exceptions in the VW slim codepath when that is incorporated.  
4. We avoid new exceptions in the example handling path and have a goal of removing any others that exist. 
5. Where error states are unavoidable (i.e. situations like arguments-don’t-make-sense) we’ll accept off-critical-path exceptions.  A proposal for refactoring around return codes is welcome, but that will need to be driven by Rajan and is subject to priorities.  At the library interface level, I believe this can be handled by having an explicit catch in the library interface for the setup calls.

# Style
1. There is a [`.clang-format file`](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/.clang-format), which outlines the general style
2. Class member variables should be prefixed with `_`. Example: `uint32_t _number_of_actions;`
# Improvements

This is a list of improvements that we want to make to the code.  Any help implementing them is of course welcome.  

### Speed/IO optimizations
- Change the io_buf structure to run in it's own thread.  Currently, reading bits into program space operates synchronously with parsing which implies that delays in the return of read() delay parsing.  This should speedup all input forms (daemon, stdin, file)
- Change the text parser to work in a read-once fashion.  Currently, input strings are read multiple times.
- There should be a way to use the algorithms as a library.

### Algorithmic improvements
- Alternate learning algorithms.  We have basic matrix factorization, which needs to be developed further.  We also want to push into more complex nonconvex algorithms.
- Learning reductions.  Previously, we've used VW as a library to implement learning reductions against, but adding a layer of abstraction in the system allowing reductions to directly operate should be doable, and desirable.  Especially in a cluster parallel environment, directly supporting learning reductions appears superior to a library implementation.

