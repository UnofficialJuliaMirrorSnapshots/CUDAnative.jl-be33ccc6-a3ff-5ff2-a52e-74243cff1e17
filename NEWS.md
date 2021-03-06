CUDAnative v1.0 release notes
=============================

This document describes major features and user-facing changes to CUDAnative.


New features
------------

  * `@device_code_...` macros make it easy to inspect generated device code even
    if the outermost function call isn't a `@cuda` invocation. This is especially
    useful in combination with, e.g., CuArrays. The `@device_code` macro dumps
    _all_ forms of intermediate code to a directory, for easy inspection ([#147]).

  * Fast versions of CUDA math intrinsics are now wrapped ([#152]).

  * Support for loading values through the texture cache, aka. `__ldg`, has been
    added. No `getindex`-based interfaced is available yet, manually use
    `unsafe_cached_load` instead ([#158]).

  * Multiple devices are supported, by calling `device!` to switch to another
    device. The CUDA API is now also initialized lazily, so be sure to call
    `device!` before performing any work to avoid allocating a context on device
    0 ([#175]).

  * Support for object and closure kernel functions has been added ([#176]).

  * IR transformation passes have been introduced to rewrite exceptions, where
    possible, to generate user-friendly messages as well as prevent hitting
    issues in `ptxas` ([#241]).

  * Code generated by `@cuda` can now be recreated manually using a low-level
    kernel launch API. The kernel objects used in that API are useful for
    reflecting on hardware resource usage ([#266]).

  * A GPU runtime library has been introduced ([#303]), implementing certain functionality
    from the Julia runtime library that would previously have prevented GPU execution
    ([#314], [#318], [#321]).


Changes
-------

  * Debug info generation now honors the `-g` flag as passed to the Julia command,
    and is no longer tied to the `DEBUG` environment variable.

  * Log messages are implemented using the new Base Julia logging system. Debug
    logging can be enabled by specifying the `JULIA_DEBUG=CUDAnative` environment
    variable.

  * The syntax of `@cuda` now takes keyword arguments, eg. `@cuda threads=1
    foo(...)`, instead of the old tuple syntax. See the documentation of `@cuda`
    for a list of supported arguments ([#154]).

  * Non isbits values can be passed to a kernel, as long as they are unused. This
    makes it easier to implement GPU-versions of existing functions, without
    requiring a different method signature ([#168]).

  * Indexing intrinsics now return `Int`, so no need to convert to `(U)Int32`
    anymore. Although this might require more registers, it allows LLVM to
    simplify code ([#182]).

  * Better error messages, showing backtraces into GPU code (#189) and detecting
    common pitfalls like recursion or use of Base intrinsics (#210).

  * Debug information is now stripped from LLVM and PTX reflection functions
    ([#208], [#214]). Use the `strip_ir_metadata` (cfr. Base) keyword argument
    to disable this.

  * Error handling and reporting has been improved. This includes
    GPU-incompatible `ccall`s which are now detected and decoded by the IR
    validator ([#248]).

  * A callback mechanism has been introduced to inform downstream users about
    device switches ([#226]).

  * Adapt.jl is now used for host-device argument conversions ([#269]).


Deprecations and removals
-------------------------

  * `CUDAnative.@profile` has been removed, use `CUDAdrv.@profile` with a manual
    warm-up step instead.

  * The `KernelWrapper` has been removed since it prevented inferring varargs
    functions ([#254]).

  * Support for `CUDAdrv.CuArray` has been removed, the CuArrays.jl package should be used
    instead ([#284]).