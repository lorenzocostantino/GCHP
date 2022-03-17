
.. _installing_with_spack:

Building Dependencies with Spack
================================

The `Spack Package Manager <https://spack.io/>`__ may be used to download and build GCHP and all of its required external software depenencies, including
CMake, MPI, ESMF, and NetCDF libraries. The only essential prerequisite for using Spack is a local C/C++/Fortran Compiler such as `GNU Compiler Collection <https://gcc.gnu.org/>`__.
You can use this local compiler to later install a different compiler version through Spack. **GCHP requires GNU Compilers version ≥ 8.3 or Intel compilers version ≥ 18.0.5**.
You must install a newer compiler through Spack if your pre-installed compiler does not meet these requirements.

There are 3 stages to setting up GCHP and/or its dependencies with Spack: setting up Spack and configuring existing libraries, installing GCHP and/or its dependencies,
and setting up an environment file for loading installed Spack packages.

All Spack users should first read the section on setting up Spack and configuring Spack to recognize existing dependencies on your system that you would like Spack to use
for building GCHP or other dependencies (such as ESMF or Slurm):

* `Setting up Spack and existing dependencies <#setting-up-spack-installing-new-compilers-and-specifying-preinstalled-dependencies>`__

You can optionally read about the most important Spack commands and tips on how they work:

* `Spack syntax and handy commands <#spack-syntax-and-handy-commands>`__

Read one of the following sections depending on which portions of the GCHP software stack you want to install using Spack:

* `Install individual software dependencies <#installing-individual-dependencies-with-spack>`__
* `Install all dependencies in one command <#one-line-install-of-gchp-dependencies-with-spack>`__
* `Install both GCHP and all of its dependencies in one command <#one-line-install-of-gchp-and-its-dependencies-with-spack>`__


Read how to load Spack libraries in an environment file for running or building GCHP or ESMF:

* `Loading Spack libraries for use with GCHP and/or ESMF <#loading-spack-libraries-for-use-with-gchp-and-or-esmf>`__



Setting up Spack, installing new compilers, and specifying preinstalled dependencies
------------------------------------------------------------------------------------


To begin using Spack, clone the latest version by typing ``git clone https://github.com/spack/spack.git``.
Execute the following commands to initialize Spack's environment (replacing ``/path/to/spack`` with the path of your `spack` directory). 
Add these commands to an environment initialization script for easy reuse.

.. code-block:: console

   $ export SPACK_ROOT=/path/to/spack
   $ . $SPACK_ROOT/share/spack/setup-env.sh


If you do not already have a copy of your preferred text editor, you can use Spack to install and load one before proceeding (e.g. ``spack install emacs; spack load emacs``). 


Installing compilers
********************

Ensure Spack recognizes any existing compiler on your system by typing ``spack compilers``. You can use this compiler to build a new one. 

Setting up and using GNU compilers
##################################

GNU compilers are free, open source, and easily installable through Spack. Execute the following at the command prompt to install version 9.3.0 of the GNU Compiler Collection:


.. code-block:: console

   $ spack install gcc@9.3.0
   

The installation of ``gcc`` may take a long time. Once it is complete, you'll need to add it to Spack's list of compilers using the following command:

.. code-block:: console


   $ spack compiler find $(spack location -i gcc@9.3.0)


Make sure that spack recognizes your new compiler by typing ``spack compilers``, which should display a list of compilers including GCC 9.3.0.


Setting up and using Intel compilers
####################################

If you would like to use Intel compilers, whether pre-installed on your system or which you would like to install through Spack using an existing license,
follow the instructions for `pre-installed Intel compilers <https://spack.readthedocs.io/en/latest/build_systems/intelpackage.html#integration-of-intel-tools-installed-external-to-spack>`__
or for `Intel compilers you want to install through Spack <https://spack.readthedocs.io/en/latest/build_systems/intelpackage.html#installing-intel-tools-within-spack>`__ on the official Spack documentation site.
In the instructions below, simply replace ``%gcc@9.3.0`` with ``%intel`` to use your Intel compilers to build libraries.


Specifying preinstalled dependencies
************************************

Just as Spack can use an existing compiler on your system, it can also use your existing installations of dependencies for GCHP rather than building new copies.
**This is essential for interfacing your cluster's Slurm with a Spack-built GCHP and its dependencies**. For any preinstalled dependency you want Spack to always use, 
you must specify its path on your system and that you want Spack to always use this preinstalled package rather than building a new version.
The code below shows how to do this by editing ``$HOME/.spack/packages.yaml`` (you may need to create this file):

.. code-block:: yaml

packages:
 slurm:
  buildable: false
  externals:
  - spec: slurm
    prefix: /path/to/slurm


Finding and specifying your system's Slurm installation
*******************************************************

If you plan on submitting jobs through the Slurm job scheduler, you'll need to specify the location of Slurm on your system for Spack.
Spack expects a specific directory format for your external Slurm installation path: it must contain both an ``include/`` directory and a ``lib64/`` directory.
Depending on your cluster's Slurm configuration, these directories may or may not already be in the same parent directory. Additionally, finding these individual directories can
prove challenging. Each of the following commands can help lead you to the correct ``include/`` and ``lib64/`` directories on your cluster:

.. code-block:: console

   $ whereis slurm
   $ whereis libpmi
   $ whereis libpmi2
   $ whereis srun
   $ whereis sinfo
   
You may or may not receive any output from each of these commands, but hopefully at least one of these commands reveals a high level Slurm directory (e.g. ``opt/slurm``),
an ``include/`` directory, and/or a ``lib64/`` directory.  You may encounter multiple directories with the name ``include`` or ``lib64``; the correct ``include/`` directory should contain
``.h`` files like ``pmi.h`` and ``slurm.h``, while the ``lib64/`` directory should contain ``libpmi.so``, ``libpmi2.so``, and/or ``libpmix.so``. 

If you have a high level Slurm directory that contains correct ``include/`` and ``lib64/`` directories, then you
can use the path to that high level directory in your ``$HOME/.spack/packages.yaml`` file.


If your revealed ``include/`` and ``lib64/`` directories are not located in the same parent directory, you'll need to create a new directory (called ``slurm_hub`` in this example,
though you can name it anything and put it anywhere) with symlinks to your ``include/`` and ``lib64/`` directories:

.. code-block:: console

   $ mkdir slurm_hub
   $ cd slurm_hub
   $ ln -s /path/to/include include
   $ ln -s /path/to/lib64 lib64
   
Put the path to ``slurm_hub`` in the correct location of your ``$HOME/.spack/packages.yaml`` file, and Spack will consider this directory to be your Slurm directory.


Spack syntax and handy commands
-------------------------------

This section describes the most important parts of Spack's syntax as well as various useful Spack commands.


When describing a Spack package, the ``@version`` syntax specifies the package version (e.g. ``gchp@13.0.2``). If you do not specify a specific version of a package
during installation, Spack will either install the latest available version of that package or Spack will choose a version that it knows to be compatible with other
dependencies in your software stack.


The ``package1 ^package2`` syntax tells Spack that ``package2`` is a dependency of ``package1``. Packages will often have locked dependency requirements such that you cannot
add totally new packages as dependencies or avoid using specific packages, but you can still use this syntax to specify information on how to build certain dependencies 
(e.g. ``spack install gchp ^netcdf-fortran@4.5.2%gcc@9.3.0``).
Other dependency requirements can be fulfilled by multiple packages (e.g. you can specify an MPI implementation to use through ``spack install gchp ^openmpi`` or ``spack install gchp ^intel-mpi``).
If you want to specify build options for a package, make sure that option immediately follows the package's name in a Spack command (e.g. ``spack install gchp +ofi ^openmpi +pmi``
is correct because ``+ofi`` is an option for ``gchp`` and ``+pmi`` is an option for ``openmpi``, but ``spack install gchp ^openmpi +pmi +ofi`` will fail because ``+ofi`` is not an option
for ``openmpi``).


The ``%compiler`` syntax specifies which compiler to use to build a package. You can specify the compiler version as well (e.g. ``gchp%gcc@9.3.0``). Spack should build all child
dependencies of a package with the same compiler you specify for the parent package, but compiler versions may or may not be consistent if you first specify a compiler version
further down the dependency tree (e.g. ``spack install gchp%gcc@9.3.0 ^openmpi`` will build all dependencies of GCHP with ``gcc@9.3.0`` including OpenMPI, 
but ``spack install gchp ^openmpi%gcc@9.3.0`` only requires Spack to build OpenMPI and its dependencies with ``gcc@9.3.0`` so other dependencies of GCHP may be built with another 
compiler version).


Every build of a package through Spack receives its own unique hash identifier. This hash can be useful when you install multiple copies of the same package that are not readily
distinguishable (e.g. they only differ by one dependency version). You can use the ``-l`` option in ``spack find`` to see the beginning of the hash ID for each package. You only need
this short code (rather than the full 32 character hash) to identify a package for Spack. Use ``/hash_id`` to specify a package build based on its hash (you don't need to
include the package's name, e.g. a GCHP installation with the hash ``2qmcc6kq3hwm3vasey63y6h4l77pzw2o`` can be loaded using ``spack load /2qmcc6``).


All of the above syntax examples can be added to any of the commands below for any ``package_name``.



``spack info package_name`` will return information on package ``package_name``, including available versions and build variant options, if this package is available through Spack.
This info is pulled from data in your clone of the Spack Git repository. You can get the latest information (as well as any updates to packages) 
by updating your Spack repo using ``git pull``.


``spack spec package_name`` will return a list of all dependencies (including package versions, compilers, and build variants) that would be installed if you were to
use ``spack install package_name``. 


``spack install package_name`` is the core Spack installation command. Many examples of ``install`` commands are found in other sections on this page. One important option for
``spack install`` is ``--only dependencies``, which installs all dependencies for a package without installing the package itself. Note that if you use ``spack install --only dependencies package_name``
you will not be able to later load all dependencies for ``package_name`` using ``spack load package_name``; you will have to load dependencies individually.


``spack uninstall package_name`` lets you uninstall specific packages. You'll be prompted with additional options to append after ``uninstall`` if the package you're attempting to remove
has dependents remaining on your system. ``spack uninstall -a pattern`` can be used to uninstall all packages matching a certain spec pattern (e.g. ``spack uninstall %gcc@9.3.0`` will uninstall
all packages that were compiled with ``gcc@9.3.0``).


``spack compiler remove compiler_name`` will remove a compiler from Spack's list of available compilers. 


``spack find package_name`` will show you all installed versions of ``package_name``. ``spack find`` will show you all installed packages, and ``spack find --loaded`` will show you all loaded
packages. 


``spack location -i package_name`` will return the installation path of ``package_name``.


``spack load package_name`` will load a package and all of its runtime dependencies.


``spack unload package_name`` will unload a specific package. Note that some environment variable changes will not be undone, and dependencies will not be unloaded.
``spack unload`` (without specifying a package) will unload all loaded packages, similar to ``module purge``. 


Installing individual dependencies with Spack
---------------------------------------------

This section describes how to use Spack to build GCHP's individual dependencies. While these dependencies can be used to then install GCHP directly using Spack,
this section is mainly intended for those looking to manually download and compile GCHP as described in the User Guide.


Installing basic dependencies
*****************************


You should now install Git and CMake using Spack:

.. code-block:: console


   $ spack install git@2.17.0%gcc@9.3.0
   $ spack install cmake@3.16.1%gcc@9.3.0


Installing without Slurm support
################################

If you do not intend to use a job scheduler like Slurm to run GCHP, use the following commands to install MPI and NetCDF-Fortran. 
Otherwise, scroll down to see necessary modifications you must make to include Slurm support.


**OpenMPI**

.. code-block:: console

      $ spack install openmpi@4.0.4%gcc@9.3.0
      $ spack install netcdf-fortran%gcc@9.3.0 ^netcdf-c^hdf5^openmpi@4.0.4


**Intel MPI**

.. code-block:: console

   $ spack install intel-mpi%gcc@9.3.0
   $ spack install netcdf-fortran%gcc@9.3.0 ^intel-mpi



**MVAPICH2**

.. code-block:: console

   $ spack install mvapich2%gcc@9.3.0
   $ spack install netcdf-fortran%gcc@9.3.0 ^netcdf-c^hdf5^mvapich2

 

Installing with Slurm support
#############################


**OpenMPI**

You need to tell Spack to build OpenMPI with Slurm support and to build NetCDF-Fortran with the correct OpenMPI version as a dependency:

.. code-block:: console

   $ spack install openmpi@4.0.4%gcc@9.3.0 +pmi schedulers=slurm
   $ spack install netcdf-fortran%gcc@9.3.0  ^netcdf-c^hdf5^openmpi@4.0.4+pmi schedulers=slurm



You may run into issues building OpenMPI if your cluster has preexisting versions of PMIx that are newer than OpenMPI's internal version. 
OpenMPI will search for and use the newest version of PMIx installed on your system, which will likely cause a crash during build because OpenMPI requires you to build with the same libevent library as was used to build PMIx. 
This information may not be readily available to you, in which case you can tweak the build arguments for OpenMPI to always use OpenMPI's internal version of PMIx. 
Open ``$SPACK_ROOT/var/spack/repos/builtin/packages/openmpi/package.py`` and navigate to the ``configure_args()`` function. In the body of this function, place the following line:

.. code-block:: python

      config_args.append('--with-pmix=internal')




**Intel MPI**

No build-time tweaks need to be made to install Intel MPI with Slurm support. 

.. code-block:: console

   $ spack install intel-mpi%gcc@9.3.0
   $ spack install netcdf-fortran%gcc@9.3.0 ^intel-mpi


Scroll down to find environment variables you need to set when running GCHP with Intel MPI, including when using Slurm.

**MVAPICH2**

Like OpenMPI, you must specify that you want to build MVAPICH2 with Slurm support and build NetCDF-Fortran with the correct MVAPICH2 version.

.. code-block:: console

   $ spack install mvapich2%gcc@9.3.0 process_managers=slurm
   $ spack install netcdf-fortran%gcc@9.3.0 ^netcdf-c^hdf5^mvapich2



Once you've installed all of your dependencies, you can follow the GCHP instructions for downloading, compiling, and setting up a run directory in the User Guide
section of this Read The Docs site.

One-line install of GCHP dependencies with Spack
------------------------------------------------


Rather than using Spack to install individual dependencies, you can use the ``spack install --only dependencies gchp`` command to install every
dependency for GCHP in a single command. The ``--only dependencies`` option tells Spack to build GCHP's dependencies without building GCHP itself.


Spack is smart about choosing compatible versions for all of GCHP's different dependencies. You can further specify which package versions or MPI
implementations (OpenMPI, Intel MPI, or MVAPICH2) you wish to use by appending options to ``spack install --only dependencies gchp``, such as ``^openmpi@4.0.4`` or ``^intel-mpi``.
If you wish to use Slurm with GCHP and want Spack to install a new version of OpenMPI or MVAPICH2, you need to specify ``+pmi schedulers=slurm`` (for OpenMPI) or ``process_managers=slurm``
(for MVAPICH2). A full install line for all of GCHP's dependencies, including Slurm-enabled OpenMPI, would look like ``spack install --only dependencies gchp ^openmpi +pmi schedulers=slurm``.


Once you've installed all of your dependencies, you can follow the GCHP instructions for downloading, compiling, and setting up a run directory in the User Guide
section of this Read The Docs site.

One-line install of GCHP and its dependencies with Spack
--------------------------------------------------------


You can use Spack to install all of GCHP's dependencies and GCHP itself in a single line: ``spack install gchp``. Just as when installing only GCHP's dependencies, you
can modify this command with further options for GCHP's dependencies (and should do so if you intend to use a job scheduler like Slurm).

Spack is smart about choosing compatible versions for all of GCHP's different dependencies. You can further specify which package versions or MPI
implementations (OpenMPI, Intel MPI, or MVAPICH2) you wish to use by appending options to ``spack install gchp``, such as ``^openmpi@4.0.4`` or ``^intel-mpi``.
If you wish to use Slurm with GCHP and want Spack to install a new version of OpenMPI or MVAPICH2, you need to specify ``+pmi schedulers=slurm`` (for OpenMPI) or ``process_managers=slurm``
(for MVAPICH2). A full install line for GCHP and all of its dependencies, including Slurm-enabled OpenMPI, would look like ``spack install gchp ^openmpi +pmi schedulers=slurm``.

In addition to specifying options for GCHP's dependencies, GCHP also has its own options you can specify in your ``spack install gchp`` command. The available options 
(which you can view for yourself using ``spack info gchp``) include:


* ``apm``          - APM Microphysics (Experimental) (Default: off)
* ``build_type``   - Choose CMake build type (Default: RelWithDebInfo)
* ``ipo``          - CMake interprocedural optimization (Default: off)
* ``luo``          - Luo et al 2019 wet deposition scheme (Default: off)
* ``omp``          - OpenMP parallelization (Default: off)
* ``real8``        - REAL\*8 precision (Default: on)
* ``rrtmg``        - RRTMG radiative transfer model (Default: off)
* ``tomas``        - TOMAS Microphysics (Experimental) (Default: off)


To specify any of these options, place it directly after ``gchp`` with a ``+`` to enable it or a ``~`` to disable it (e.g. ``spack install gchp ~real8 +rrtmg``).


When you run ``spack install gchp``, Spack will build all of GCHP's dependencies and then download and build GCHP itself. The overall process may take a very long time if you
are installing fresh copies of many dependencies, particularly MPI or ESMF. Once the install is completed, Spack will leave you with a built ``gchp`` executable and a copy of GCHP's
source code at ``spack location -i gchp``. 


You can use Spack's included copy of the source code to create a run directory. Navigate to the directory returned by ``spack location -i gchp``, and then ``cd`` to ``source_code/run``.
Run ``./createRunDir.sh`` to generate a GCHP run directory. Once you've created a run directory, follow the `instructions on Running GCHP in the User Guide <../user-guide/running.html>`__.

You can find information on loading your environment for running GCHP below.



Loading Spack libraries for use with GCHP and/or ESMF
-----------------------------------------------------

After installing the necessary libraries, place the following in a script that you will run before building/running GCHP (such as ``$HOME/.bashrc`` or a separate environment script)
to initialize Spack and load requisite packages for building ESMF and/or building/running GCHP.


**OpenMPI**

.. code-block:: bash

    export SPACK_ROOT=$HOME/spack #your path to Spack
    source $SPACK_ROOT/share/spack/setup-env.sh
    if [[ $- = *i* ]] ; then
     echo "Loading Spackages, please wait ..."
    fi
    #==============================================================================
    %%%%% Load Spackages %%%%%
    #==============================================================================
    # List each Spack package that you want to load
    # NOTE: Only needed if you did not install GCHP directly through Spack
    pkgs=(gcc@9.3.0       \
     git@2.17.0           \
     netcdf-fortran@4.5.2 \
     cmake@3.16.1         \
     openmpi@4.0.4        \
     esmf@8.0.1           )

    # Load each Spack package
    for f in ${pkgs[@]}; do
      echo "Loading $f"
      spack load $f
    done
    
    # If you installed GCHP directly through Spack,comment out the above code after "Load Spackages"
    # and uncomment the following line
    #spack load gchp
    
    export MPI_ROOT=$(spack location -i openmpi)
    
    # These lines only needed for building ESMF outside of Spack
    export ESMF_COMPILER=gfortran #intel for intel compilers
    export ESMF_COMM=openmpi

**Intel MPI**

.. code-block:: bash

    export SPACK_ROOT=$HOME/spack #your path to Spack
    source $SPACK_ROOT/share/spack/setup-env.sh
    if [[ $- = *i* ]] ; then
     echo "Loading Spackages, please wait ..."
    fi
    #==============================================================================
    %%%%% Load Spackages %%%%%
    #==============================================================================
    # List each Spack package that you want to load
    # NOTE: Only needed if you did not install GCHP directly through Spack
    pkgs=(gcc@9.3.0       \
     git@2.17.0           \
     netcdf-fortran@4.5.2 \
     cmake@3.16.1         \
     intel-mpi            \
     esmf                 )

    # Load each Spack package
    for f in ${pkgs[@]}; do
      echo "Loading $f"
      spack load $f
    done
    
    # If you installed GCHP directly through Spack,comment out the above code after "Load Spackages"
    # and uncomment the following line
    #spack load gchp

    # Environment variables only needed for Intel MPI
    export I_MPI_CC=gcc #icc for intel compilers
    export I_MPI_CXX=g++ #icpc for intel compilers
    export I_MPI_FC=gfortran #ifort for intel compilers
    export I_MPI_F77=gfortran #ifort for intel compilers
    export I_MPI_F90=gfortran #ifort for intel compilers
    export MPI_ROOT=$(spack location -i intel-mpi)

    export I_MPI_PMI_LIBRARY=/path/to/slurm/libpmi2.so #when using srun through Slurm
    #unset I_MPI_PMI_LIBRARY #when using mpirun

    # These lines only needed for building ESMF outside of Spack
    export ESMF_COMPILER=gfortran #intel for intel compilers
    export ESMF_COMM=intelmpi
    
    


**MVAPICH2**

.. code-block:: bash

    export SPACK_ROOT=$HOME/spack #your path to Spack
    source $SPACK_ROOT/share/spack/setup-env.sh
    if [[ $- = *i* ]] ; then
     echo "Loading Spackages, please wait ..."
    fi
    #==============================================================================
    %%%%% Load Spackages %%%%%
    #==============================================================================
    # List each Spack package that you want to load
    # NOTE: Only needed if you did not install GCHP directly through Spack
    pkgs=(gcc@9.3.0       \
     git@2.17.0           \
     netcdf-fortran@4.5.2 \
     cmake@3.16.1         \
     mvapich2             \
     esmf                 )

    # Load each Spack package
    for f in ${pkgs[@]}; do
      echo "Loading $f"
      spack load $f
    done
    
    # If you installed GCHP directly through Spack,comment out the above code after "Load Spackages"
    # and uncomment the following line
    #spack load gchp
    
    export MPI_ROOT=$(spack location -i mvapich2)
    
    # These lines only needed for building ESMF outside of Spack
    export ESMF_COMPILER=gfortran #intel for intel compilers
    export ESMF_COMM=mvapich2
    

You can also add other packages you've installed with Spack like ``emacs`` to the ``pkgs`` lists above.


ESMF and your environment file
------------------------------

The following gives some information on building ESMF separately from Spack and provides more environment file examples.


You must load your environment file prior to building and running GCHP.

.. code-block:: console

   $ source /home/envs/gchpctm_ifort18.0.5_openmpi4.0.1.env

If you don't already have ESMF 8.0.0+, you will need to download and build it. You only need to
build ESMF once per compiler and MPI configuration (this includes for ALL users on a cluster!). It
is therefore worth downloading and building somewhere stable and permanent, as almost no users of
GCHP would be expected to need to modify or rebuild ESMF except when adding a new compiler or MPI.
ESMF is available through Spack, and will already be installed if you chose the
``spack install gchp --only dependencies`` or ``spack install gchp`` routes above.
Instructions for manually downloading and building ESMF are available at the GCHP wiki.

It is good practice to store your environment setup in a text file for reuse. Below are a couple
examples that load libraries and export the necessary environment variables for building and running
GCHP. Note that library version information is included in the filename for easy reference. Be sure
to use the same libraries that were used to create the ESMF build install directory stored in
environment variable :envvar:`ESMF_ROOT`.

**Environment file example 1**

.. code-block:: bash

   # file: gchpctm_ifort18.0.5_openmpi4.0.1.env

   # Start fresh
   module --force purge

   # Load modules (some include loading other libraries such as netcdf-C and hdf5)
   module load intel/18.0.5
   module load openmpi/4.0.1
   module load netcdf-fortran/4.5.2
   module load cmake/3.16.1

   # Set environment variables
   export CC=gcc
   export CXX=g++
   export FC=ifort

   # Set location of ESMF
   export ESMF_ROOT=/n/lab_shared/libraries/ESMF/ESMF_8_0_1/INSTALL_ifort18_openmpi4

**Environment file example 2 (Spack libraries built with a pre-installed compiler)**

.. code-block:: bash

   # file: gchpctm_gcc7.4_openmpi.rc

   # Start fresh
   module --force purge

   # Load modules
   module load gcc-7.4.0
   spack load cmake
   spack load openmpi%gcc@7.4.0
   spack load hdf5%gcc@7.4.0
   spack load netcdf%gcc@7.4.0
   spack load netcdf-fortran%gcc@7.4.0

   # Set environment variables
   export CC=gcc
   export CXX=g++
   export FC=gfortran

   # Set location of ESMF
   export ESMF_ROOT=/n/home/ESMFv8/DEFAULTINSTALLDIR
