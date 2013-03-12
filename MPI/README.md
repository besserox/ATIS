# UL HPC Tests for MPI Modules

## Synopsis

This folder proposes a set of unit tests to check basic functionality of an MPI stack.
The unit tests are started with CTest and results can be submitted to the UL CDash.
The configuration of the script is managed by CMake.

[UL CDash](http://cdash.uni.lux/index.php?project=Modules-Check)

## Configure the scripts


* Checkout the sources

    $> git clone git@github.com:ULHPC/modules-check.git modules-check-src


* Create a directory to hold the configured scripts

    $> mkdir modules-check-MPI


* Generate the script with CMake

    $> cd modules-check-MPI  
    $> cmake ../modules-check-src


## Testing an MPI stack


* Running the tests

    $> ctest

  This run the test with the current environment.


* Running the tests for a given module

    $> module purge           
    $> module load OpenMPI    
    $> ctest


* You can even try

    $> export PATH=/path/to/my/mpi/install:$PATH
    $> ctest



## Submitting test results

* Test results can be viewed on [UL CDash](http://cdash.uni.lux/index.php?project=Modules-Check)


* ctest allows to submit the results to the UL dashboard in the Experimental group

    $> ctest -M Experimental


* Alternatively, you can use the test\_and\_submit.cmake script to run and submit the results with explicit names

    $> export CDASH\_GROUP\_NAME="Experimental" CDASH\_SITE\_NAME="Gaia cluster" CDASH\_BUILD\_NAME="MPI module 'OpenMPI'"  
    $> ctest -S test\_and\_submit.cmake



## Automated testing


* The testing of all MPI modules can be triggered interactively with the check\_all\_mpi\_modules.sh

    $> ./check\_all\_mpi\_modules.sh

  This script lists all the available MPI module, tests all of them, and submit the results to the UL dashboard.
  It must be run inside an OAR job.


* The same script can be submitted to OAR to perform non-interactive testing of all MPI modules

    $> oarsub -S ./check\_all\_mpi\_modules.sh

  This script already contains the required OAR parameters. It must be run on the cluster frontend.
