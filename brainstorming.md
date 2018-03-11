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

  * audit numpy to find the places where it's currently special-casing
    different dtypes, so they can be moved into methods.
  
    * Anything that checks `PyDataType_HASFIELDS` is probably a hack
      like this, e.g. see the use in `array_dealloc` and
      `PyArray_XDECREF`
      (note: [#10721](https://github.com/numpy/numpy/issues/10721))

    * `.real` and `.imag` might be examples

    * most likely to be seen for object dtype, void/record dtypes,
      maybe string dtypes

  * casting is probably the hardest design problem
  
    * and in particular, making ufunc loop selection fast

  * need to refactor ufuncs, so that loops get the dtype object as one
    of their arguments. (and maybe other cleanups while we're at it?)

    * need to claw back internals of ufuncs from the public API
      * work with Numba devs, since as far as we know they're the only
        users
    * need to have compatibility shims for old-style loop registration
    * need mechanism at loop level to create output dtype objects

  * categorical data
    * Contact:
      * pandas dev
      * sklearn dev (PR for categorical support already opened)
      * matplotlib dev (there is "categorical" support in matplotlib 2.1)

* missing data

  * depends on dtype refactors
  * then add some dtype method for isna, and teach ufunc loops to
    handle NAs in some general way?

* duck arrays

  * make a list of everything in numpy and work through it...
  
  * start with `__array_concatenate__`? what should it look like?
  
    * there's stack and concatenate; should they be one special method
      or two? (the only difference is whether an axis is created). I
      guess I'm leaning towards `__array_stack__ ` and
      `__array_concatenate__` and if they share a bunch of code that's
      OK. (Cf. Python's comparison special methods.)
      
      `np.concatenate` and `np.stack` have extremely simple APIs, so
      replacing them is probably pretty straightforward. In the
      implementation we'll want to do some refactoring to pull out the
      `__array_ufunc__` dispatch logic. (And that's itself in some
      weird position because of the multiarray/umath split; maybe we
      should be prioritizing fixing that?)
      
    * More complicated problem: we don't want to have
      `__array_hstack__`, `__array_vstack__`, etc.; duck array authors
      should just have to implement
      `__array_concatenate__`/`__array_stack__` and the rest should
      work automatically. So what other operations do these need?

      In current numpy, dependencies (format is `x: y` means that x is
      implemented using y):
      
      * `concatenate`: is primitive
      * `stack`: `asanyarray`, `.shape`, `.ndim`, slicing with
        `np.newaxis`, `concatenate`
      * `hstack`: `atleast_1d`, `.ndim`, `concatenate`
      * `vstack`: `atleast_2d`, `concatenate`
      * `dstack`: `atleast_3d`, `concatenate`
      * `row_stack`: `atleast_2d`, `concatenate`
      * `column_stack`: `array(v, copy=False, subok=True)`, `.ndim`,
        `array` with `ndmin=2`, `.T`, `concatenate` – this one is
        super tricky
      * `atleast_1d`: `asanyarray`, `reshape`
      * `atleast_2d`: `asanyarray`, `reshape`, slicing with `newaxis`
      * `atleast_3d`: `asanyarray`, `reshape`, slicing with `newaxis`

      So I guess the main things are some kind of solution for
      `as(any)array`, `.ndim` (not a big deal, we can just tell people
      to implement this), and inserting axes?
      
      We should ask folks like @shoyer if they have a preference for
      how the insert-an-axis operation should be spelled. Could just
      be indexing with `newaxis`, but I don't know if people actually
      like implementing that. I worry that `reshape` might be
      difficult for some implementations (e.g. distributed arrays
      where in the general case it might require an arbitrary
      cross-machine shuffle).

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

* get oindex unblocked and clean up fancy indexing

* split off numpy.ma into its own library?

* finally do the np.float etc. deprecations

* review scipy.org's hosting situation, automate doc uploads etc.

* consider experimenting with BLIS again, now that it has runtime
  auto-configuration support:
  https://groups.google.com/forum/#!topic/blis-devel/z__DDqkIikY
