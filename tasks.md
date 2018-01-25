- Merging multiarray & umath
  - Currently compiles into to libraries
  - Any functions called in both *have* to be added to public API
  - Bunch of files included in both with C-Preprocessor (icky)
  - Mostly ufuncs call ndarrays via API, but sometimes (A + B) array
    needs to call ufunc.
    - set_numericops (?)
