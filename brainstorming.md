Starting to brainstorm a big list of things that we might want to do,
so we can start thinking about prioritizing/planning?

# major features

* Dtypes

  * need to define "dtypetype" and "dtype metaclass" -- so we can add C
    slots to dtypetype's
    * including some backwards compatibility system for legacy dtypes
    * start with only the C level stuff; add Python dtype definition
      as a second pass
    * it would be good to make it easy to define new dtypes in cython
      -- but, currently cython does not understand metaclasses. So
      this may require cython extensions.

  * casting is probably the hardest design problem
  
    * and in particular, making ufunc loop selection fast

  * need to refactor ufuncs, so that loops get the dtype object as one
    of their arguments. (and maybe other cleanups while we're at it?)

    * need to claw back internals of ufuncs from the public API
      * work with Numba devs, since as far as we know they're the only
        users
    * need to have compatibility shims for old-style loop registration
    * need mechanism at loop level to create output dtype objects

* missing data

  * depends on dtype refactors
  * then add some dtype method for isna, and teach ufunc loops to
    handle NAs in some general way?

* duck arrays

  * make a list of everything in numpy and work through it...
  
  * start with `__array_concatenate__`? what should it look like?
  
  * enhance (g)ufunc API so that more of numpy's API can become
    (g)ufuncs

  * better sparse arrays in scipy, so we can deprecate np.matrix?

# smaller changes

* clarifying the NEP process (steal from PEPs?)

  * don't want useless bureaucracy, but things like the PEP template
    and general process guidelines could make things easier

  * something for N+J to work on soon, we will meet this friday
    (11/17)

* migrating some stuff into Cython

  * specifically, python->C transitions to help with things like kwarg
    parsing, refcounting, ...

  * main complication: we want to have a single extension module built
    out of multiple .c and .pyx files

    * the "public" declaration seems to do what we want
    
    * module init is going to be "interesting"
    
    * hold a cython+numpy sprint?

* merge multiarray and umath [related to the above?]

* improved benchmark tracking

  * dedicated hardware?

* more thorough/automated pre-release QA?

* better ability to change ABI

* rationalizing scipy/numpy inconsistencies between linalg, fft

* get oindex unblocked

* split off numpy.ma into its own library?

* finally do the np.float etc. deprecations

* review scipy.org's hosting situation, automate doc uploads etc.

