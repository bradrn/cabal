How to use Cabal in Windows
=================================

The Windows toolchain and environment
-------------------------------------

In general, Cabal should not require any Windows-specific configuration. The
major exception is when external C libraries are required, in which case those
libraries must be installed if not yet present, and Cabal must be informed of
their location.

The recommended method for installing C libraries is using
`MSYS2 <https://www.msys2.org/>`_, which provides a toolchain and Unix-like
environment for development on Windows. The following guide describes how to
integrate MSYS2 with Cabal.

.. note::

    Most distributions of Haskell for Windows, including
    `GHCup <https://www.haskell.org/ghcup/>`_ and
    `Haskell Stack <https://docs.haskellstack.org/en/stable/>`_,
    come with their own MSYS2 installation. If present, it is strongly
    recommended to use this environment rather than downloading another copy
    of MSYS2.

    GHC for Windows is also shipped with a MinGW toolchain included. However,
    this is not a full MSYS2 installation, and hence does not bundle necessary
    tools such as ``make`` and ``autoconf``.

    Therefore, since the installation location of MSYS2 changes with the tool used,
    this guide will refer to its path with the notation ``<MSYS>``.

Using system libraries with GHC
-------------------------------

We will begin by describing how to use MSYS2 libraries with GHC alone. Using them
with Cabal will then follow the same basic principles.

First, the desired packages must be installed in an MSYS2 environment. To do this,
start a shell within your MSYS2 installation, then use the ``pacman`` package
manager. For more details refer to `MSYS2 package management documentation
<https://www.msys2.org/docs/package-management/>`_.

.. note::
    An MSYS2 installation contains several environments. Each package has
    different versions for different environments. Both GHCup and Stack use
    the MINGW64 environment by default, and it is recommended to use that
    environment for all packages, by only installing packages with the prefix
    ``mingw64/``. Alternatively, the UCRT64 or CLANG64 environments may be
    suitable for some purposes. In the following examples, ``<environment>``
    will stand for the environment used to install packages.

Once a package has been installed, the locations in which its files have been
installed must be provided to GHC. Specifically, it requires the following
information:

* The directory in which the header files have been installed, specified with ``-I``
* The location of the library files, specified with ``-L``
* The name of the library, specified with ``-l``

For instance, if compiling a program which depends on ``libssl``, GHC should be
invoked with the following arguments:

::

   > ghc ... -I<MSYS>\\<environment>\\includes -L<MSYS>\\<environment>\\lib -lssl

This should result in a succesful compilation.

Using Cabal on Windows
----------------------

To use MSYS2 libraries in Cabal, similar configuration fields must be specified:

::

     extra-include-dirs: <MSYS>\<environment>\include
     extra-lib-dirs: <MSYS>\<environment>\lib
     extra-prog-path: <MSYS>\<environment>\bin,
                      <MSYS>\usr\bin

The first two of these correspond straightforwardly to the GHC options ``-I``
and ``-l`` respectively. ``extra-prog-path`` adds MSYS2 directories to the PATH
used for Cabal invocations, so that it can find common Unix tools (e.g. ``make``).

It is recommended to add these fields to
:ref:`the user-wide global configuration <config-file-discovery>`.
This is so the MSYS2 environment is used for every package, including those
downloaded from repositories such as Hackage or Stackage.

.. note::
    If Cabal was installed via a tool such as GHCup and Stack, the global
    configuration should have already been set up as needed.

In some cases, it may be useful to set these fields project-locally. Since
the paths may differ between Haskell installations, it is recommmended to
set them in ``cabal.project.local``, using something like the following:

::

   package *
     extra-include-dirs: <MSYS>\<environment>\include
     extra-lib-dirs: <MSYS>\<environment>\lib
     extra-prog-path: <MSYS>\<environment>\bin,
                      C:\msys64\usr\bin
