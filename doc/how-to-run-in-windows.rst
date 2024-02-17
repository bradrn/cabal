How to use Cabal in Windows
=================================

The Windows toolchain and environment
-------------------------------------

`MinGW <https://www.mingw-w64.org/>`_ is a set of software packages for the
Windows platform that enable development and compilation targeting the native
Windows API. MinGW does not offer a complete developer environment so it usually
is coupled with `MSYS2 <https://www.msys2.org/>`_ which provides many UNIX-like
tools such as ``bash``, ``make`` or the package manager ``pacman``.

.. note::

    Most distributions of Haskell for Windows, including
    `GHCup <https://www.haskell.org/ghcup/>`_ and
    `Haskell Stack <https://docs.haskellstack.org/en/stable/>`_,
    already install MSYS2 alongside the rest of Haskell.
    It is strongly recommended to use this environment rather than downloading
    another copy of MSYS2.

    GHC for Windows is also shipped with a MinGW toolchain included. However,
    it is of limited use as it does not bundle necessary tools such as ``make``
    and ``autoconf``.

    Since MSYS2 may therefore be installed in one of several locations, this
    guide will use the notation ``<MSYS>`` to refer to the path to the relevant
    MSYS2 installation.

.. _Using system libraries with GHC:

Using system libraries with GHC
-------------------------------

Packages can be installed in an MSYS2 environment via the ``pacman`` package
manager. Refer to `MSYS2 package management documentation
<https://www.msys2.org/docs/package-management/>`_ for details.

.. note::
    When working in the MSYS2 environment bundled with GHCup and Stack, it is
    recommended to only install packages with the prefix ``mingw64/``.
    Alternately, ``ucrt64/`` or ``clang64/`` may be suitable for some purposes.
    In the following examples, ``<environment>`` will refer to the prefix used.

Once the package is installed, the appropriate location of the libraries should
be given to GHC, for example if compiling something that depends on
``libssl.a``:

::

   > ghc ... -I<MSYS>\\<environment>\\includes -L<MSYS>\\<environment>\\lib -lssl

This should result in a succesful compilation.

Using Cabal on Windows
----------------------

All the above extra configurations will also be needed when using ``cabal``, and
one can provide all of them using the following Cabal configuration fields:

::

     extra-include-dirs: <MSYS>\<environment>\include
     extra-lib-dirs: <MSYS>\<environment>\lib
     extra-prog-path: <MSYS>\<environment>\bin,
                      C:\msys64\usr\bin

Note that ``extra-include-dirs: x`` translates to the GHC option ``-Ix``,
``extra-lib-dirs: x`` translates to the GHC option ``-Lx`` and
``extra-prog-path: x`` results in the path ``x`` being added to the ``PATH``
environment when calling an executable by cabal.

It is recommended to add these fields to
:ref:`the user-wide global configuration <config-file-discovery>`.
This is so the MSYS2 environment is used for every package, including those
downloaded from repositories such as Hackage or Stackage.

.. note::
    If Cabal was installed via a tool such as GHCup and Stack, the global
    configuration should have already been set up as needed.

In some cases, it may be useful to set these flags project-locally. Since
the paths may differ between Haskell installations, it is recommmended to
add the following to ``cabal.project.local``:

::

   package *
     extra-include-dirs: <MSYS>\<environment>\include
     extra-lib-dirs: <MSYS>\<environment>\lib
     extra-prog-path: <MSYS>\<environment>\bin,
                      C:\msys64\usr\bin
