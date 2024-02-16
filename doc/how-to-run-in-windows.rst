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

.. note::

    When the code snippets must be run in a MSYS2 shell, we will use the ``msys2>``
    prompt and when in a native Windows shell we will use the ``PS>`` or ``cmd>``
    prompt. If either shell will do, we will just use ``>``.

A small note on configuring and updating MSYS2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before starting with MSYS2, one should make sure it is updated and properly configured. For that:

::

   # Update the system, run twice
   msys2> pacman --noconfirm -Syuu
   msys2> pacman --noconfirm -Syuu

   # install basic utilities
   msys2> pacman --noconfirm -S --needed curl autoconf mingw-w64-ucrt-x86_64-pkgconf
   msys2> pacman --noconfirm -S ca-certificates

   # Optionally if you like zsh
   msys2> pacman --noconfirm -S zsh

   # make $HOME the same in both PowerShell and MSYS2's Bash
   msys2> sed -i -e 's/db_home:.*$/db_home: windows/' /etc/nsswitch.conf

   # Inherit Windows' path in MSYS2
   msys2> sed -i -e 's/rem set MSYS2_PATH_TYPE=inherit/set MSYS2_PATH_TYPE=inherit/' /c/msys64/msys2_shell.cmd

As a terminal, `Windows Terminal
<https://apps.microsoft.com/detail/9N0DX20HK701>`_ usually comes in handy. I use
the following profile:

::

   {
     "colorScheme": "One Half Dark",
     "commandline": "C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -ucrt64 -shell zsh",
     "hidden": false,
     "icon": "C:\\msys64\\ucrt64.ico",
     "name": "MSYS2 / UCRT64",
     "startingDirectory": "C:/Users/<YourName>"
   }

Installing a Haskell environment on Windows
-------------------------------------------

The recommended way of installing Haskell in Windows is via `GHCup
<https://www.haskell.org/ghcup/>`_. There are however two different ways of
using GHCup on Windows:

GHCup without having installed MSYS2 already
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the user just has fresh system, this option will install MSYS2 as a first
step. The command, as presented in the front page of `GHCup's website
<https://www.haskell.org/ghcup/>`_, has to be executed on a PowerShell:

::

   PS> Set-ExecutionPolicy Bypass -Scope Process -Force;[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; try { Invoke-Command -ScriptBlock ([ScriptBlock]::Create((Invoke-WebRequest https://www.haskell.org/ghcup/sh/bootstrap-haskell.ps1 -UseBasicParsing))) -ArgumentList $true } catch { Write-Error $_ }

After installing MSYS2 it will run the same script as the non-Windows systems
run with the appropriate values for the environment variables. This command will
also set the appropriate Environment Variables for Windows.

GHCup on a system that already has MSYS2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

GHCup can work with an already existing installation of MSYS2. For that, do the
following steps:

::

   msys2> export GHCUP_MSYS2=/c/msys64
   msys2> curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh

For more information about other environment variables, refer to `the GHCup
documentation <https://www.haskell.org/ghcup/guide/#env-variables>`_.

It is advisable also to set ``GHCUP_MSYS2=C:\msys64`` as a System variable on
Windows:

1. Open the search bar and search for "Edit the system environment variables"

2. Switch to the "Advanced" tab and click on "Environment Variables..."

3. Click on "New..." and fill in the values.

Additionally, in order to be able to invoke the installed tools, the ``PATH``
should be modified to include the GHCup binaries directory:

1. Open the search bar and search for "Edit the system environment variables"

2. Switch to the "Advanced" tab and click on "Environment Variables..."

3. Double click on ``PATH`` and add ``;C:\ghcup\bin`` at the end of the already existing value

Invoking GHC on Windows
-----------------------

GHC for Windows is shipped with a MinGW toolchain included. The set of included
packages can be found in `the GHC downloads page
<https://downloads.haskell.org/ghc/mingw>`_. Starting on version 0.7 of the
MinGW toolchain, the included packages are based on the ``CLANG64`` environment.
This built-in toolchain allows GHC to run on native Windows by using the bundled
``clang`` compiler.

Once GHCup installed GHC, it is possible to invoke just it from any Windows
shell (such as PowerShell or ``cmd``) as long as the ``ghc.exe`` executable is
in the ``PATH``. When running ``ghc --info``, it can be seen that GHC already
sets flags for includes and library paths for the bundled toolchain:

::

    > ghc --info
    [("Project name","The Glorious Glasgow Haskell Compilation System")
    ,("C compiler command","C:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/bin/clang.exe")
    ,("C compiler flags","--rtlib=compiler-rt -D_UCRT -Qunused-arguments -IC:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/include")
    ,("C++ compiler command","C:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/bin/clang++.exe")
    ,("C++ compiler flags","-IC:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/include")
    ,("C compiler link flags","-fuse-ld=lld --rtlib=compiler-rt -D_UCRT  -LC:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/lib -LC:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/x86_64-w64-mingw32/lib")
    ,("C compiler supports -no-pie","NO")
    ,("Haskell CPP command","C:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/bin/clang.exe")
    ,("Haskell CPP flags","-E -undef -traditional -Wno-invalid-pp-token -Wno-unicode -Wno-trigraphs -IC:\\ghcup\\ghc\\9.8.1\\lib\\../mingw/include")
    ...

However this is of limited use because GHC will only be able to invoke tools
that are bundled in the installation. It won't be able to call things like
``make`` or ``autoconf`` which are usually needed for configuring packages, and
it will also not be able to see any third-party libraries that are installed in
the system.

.. _Using system libraries with GHC:

Using system libraries with GHC
-------------------------------

Packages can be installed in an MSYS2 environment via the ``pacman`` package
manager. The installation of those packages is done in standard directories in
the system, namely:

- ``C:\msys64\usr\{bin,include,lib}``: for basic POSIX-like utilities. We might
  use tools that are in ``C:\msys64\usr\bin`` but we should not link against
  libraries in ``C:\msys64\usr\lib`` as those target ``msys-2.0.dll`` as a
  runtime dependency which is a fork of Cygwin and not a native Windows DLL.
  These are referred to as belonging to the ``msys`` repository.

  In order to use tools from ``C:\msys64\usr\bin`` we need to make sure it is
  present in the ``PATH``.

- ``C:\msys64\<environment>\{bin,include,lib}``: for tools and libraries that
  are compiled with the appropriate toolchain for that environment. We should
  provide these directories for includes and library paths if we want to use
  packages installed in the system.

As an example, see that MSYS2 Packages offer two different versions of
``openssl``, one for `msys <https://packages.msys2.org/base/openssl>`_ and one
for `every other environment
<https://packages.msys2.org/base/mingw-w64-openssl>`_ (and in particular for
CLANG64 or UCRT64). When installing packages from MSYS2, one should pick the
options that show ``ucrt64/`` or ``clang64/`` as prefixes, and avoid the ones
that show ``msys/``.

::

   msys2> pacman -Ss openssl
   ucrt64/mingw-w64-ucrt-x86_64-openssl 3.2.0-1
       The Open Source toolkit for Secure Sockets Layer and Transport Layer Security (mingw-w64)
   ...
   clang64/mingw-w64-clang-x86_64-openssl 3.2.0-1
       The Open Source toolkit for Secure Sockets Layer and Transport Layer Security (mingw-w64)
   ...
   msys/openssl 3.2.0-1
       The Open Source toolkit for Secure Sockets Layer and Transport Layer Security

   msys2> pacman -S ucrt64/mingw-w64-ucrt-x86_64-openssl

Once the package is installed, the appropriate location of the libraries should
be given to GHC, for example if compiling something that depends on
``libssl.a``:

::

   > ghc ... -IC:\\msys64\\ucrt64\\includes -LC:\\msys64\\ucrt64\\lib -lssl

This should result in a succesful compilation.

Using UNIX-like binaries with GHC
---------------------------------

As said above, the binaries that we might need to use are (by default) located
in two places: ``C:\msys64\usr\bin`` and ``C:\msys64\<environment>\bin``. The
easiest way to give GHC access to these executables is by including those
directories in your ``PATH``.

- On PowerShell:
  ::

     PS> $env:Path += ';C:\msys64\usr\bin;C:\msys64\ucrt64\bin'

     PS> (Get-Command make.exe).Path
     C:\msys64\usr\bin\make.exe

     PS> make --version
     GNU Make 4.4.1
     Built for x86_64-pc-msys
     ...

- On ``cmd``:
  ::

     cmd> set PATH=%PATH%;C:\msys64\usr\bin;C:\msys64\ucrt64\bin

     cmd> where make
     C:\msys64\usr\bin\make.exe

     cmd> make --version
     GNU Make 4.4.1
     Built for x86_64-pc-msys
     ...

- On MSYS2 this is usually already set (otherwise your whole system is probably
  broken), but just in case one would do the following:

  ::

     msys2> export PATH=$PATH:/c/msys64/usr/bin:/c/msys64/ucrt64/bin

     msys2> which make
     /usr/bin/make

     msys2> make --version
     GNU Make 4.4.1
     Built for x86_64-pc-msys
     ...

If one really wants, it is possible to edit the system ``PATH`` to add these two
directories (although as explained below, Cabal will automatically add them for
us) by doing:

1. Open the search bar and search for "Edit the system environment variables"

2. Switch to the "Advanced" tab and click on "Environment Variables..."

3. Double click on ``PATH`` and add ``;<dir1>;<dir2>`` at the end of the already existing value

Once these directories are in the path, any UNIX-like utilities we might need
will be in scope, so configuring packages should be able to find them.

Using Cabal on Windows
----------------------

Cabal will also be installed automatically by GHCup. Once the location of the
binary is present in the ``PATH``, ``cabal`` can be invoked from any Windows
shell, just as GHC.

::

   > cabal --help
   Command line interface to the Haskell Cabal infrastructure.

   See http://www.haskell.org/cabal/ for more information.

   Usage: cabal-3.10.2.1.exe [GLOBAL FLAGS] [COMMAND [FLAGS]]

   ...

As both ``cabal.exe`` and ``ghc.exe`` are placed in the same location by GHCup,
normally cabal will be able to invoke GHC as said location will be in the
``PATH``.

All the above extra configurations will also be needed when using ``cabal``, and
one can provide all of them via the following local project file:

::

   msys2> cat << EOF > cabal.project.local
   package *
     extra-include-dirs: C:\msys64\ucrt64\include
     extra-lib-dirs: C:\msys64\ucrt64\lib
     extra-prog-path: C:\msys64\ucrt64\bin,
                      C:\msys64\usr\bin
   EOF

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

In there, just add the lines mentioned above. For even better user experience,
it is advisable to add also two additional directories to the
``extra-prog-path`` section:

- ``C:\ghcup\bin``: in the unexpected case that said directory is not in the
  ``PATH`` already (which would mean that ``cabal.exe`` was invoked via its full
  path or that it is not in the standard location).

- ``C:\Users\<YourName>\AppData\Roaming\cabal\bin`` or whichever directory cabal
  installs binaries, so that other Haskell executables are also findable by
  cabal-invoked tools. See whether the ``installdir:`` section is uncommented
  and use its value.

The configuration would then look like this:

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
