Haskell, GHC and Cabal in Windows
=================================

The Windows toolchain and environment
-------------------------------------

`MinGW <https://www.mingw-w64.org/>`_ is a set of software packages for the
Windows platform that enable development and compilation targeting the native
Windows API. MinGW does not offer a complete developer environment so it usually
is coupled with `MSYS2 <https://www.msys2.org/>`_ which provides many UNIX-like
tools such as ``bash``, ``make`` or the package manager ``pacman``.

MSYS2 comes with different environments. You can learn more about the different
environments in the `MSYS2 documentation
<https://www.msys2.org/docs/environments/>`_. We will be using ``UCRT64`` in the
examples below but it can be swapped by ``CLANG64`` (as long as the external
packages you want to use are available for the ``CLANG64`` environment, see
:ref:`Using system libraries with GHC`).
In the following examples, ``<environment>`` will refer to the environment used.

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

Once the package is installed, the appropriate location of the libraries should
be given to GHC, for example if compiling something that depends on
``libssl.a``:

::

   > ghc ... -I<MSYS>\\<environment>\\includes -L<MSYS>\\<environment>\\lib -lssl

This should result in a succesful compilation.

Using Cabal on Windows
----------------------

All the above extra configurations will also be needed when using ``cabal``, and
one can provide all of them via the following local project file:

::

   package *
     extra-include-dirs: <MSYS>\<environment>\include
     extra-lib-dirs: <MSYS>\<environment>\lib
     extra-prog-path: <MSYS>\<environment>\bin,
                      C:\msys64\usr\bin

Note that ``extra-include-dirs: x`` translates to the GHC option ``-Ix``,
``extra-lib-dirs: x`` translates to the GHC option ``-Lx`` and
``extra-prog-path: x`` results in the path ``x`` being added to the ``PATH``
environment when calling an executable by cabal.

However, this has the inconvenience that we would need to set this up for every
project that we want to build, and we sometimes wouldn't be able to install
binaries from Hackage directly as there is no local project to refer to.

The recommended way of setting this is then modifying the global cabal
configuration, whose location you can find by calling ``cabal --help``:

::

   > cabal --help
   ...
   You can edit the cabal configuration file to set defaults:
      C:\Users\<YourName>\AppData\Roaming\cabal\config

In there, just add the lines mentioned above.
::

   > cat C:\\Users\\<YourName>\\AppData\\Roaming\\cabal\\config
   ...
   extra-include-dirs: C:\msys64\ucrt64\include
   ...
   extra-lib-dirs: C:\msys64\ucrt64\lib
   ...
   extra-prog-path: C:\ghcup\bin,
                    C:\Users\<YourName>\AppData\Roaming\cabal\bin,
                    C:\msys64\ucrt64\bin,
                    C:\msys64\usr\bin
   ...


.. note::
    If Cabal was installed via a tool such as GHCup and Stack, this file
    should have already been configured as needed.
