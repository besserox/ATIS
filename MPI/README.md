# UL HPC Tests for MPI Modules

## Synopsis

This folder proposes a set of unit tests to check basic functionality of an MPI stack.
The unit tests are started with CTest and results can be submitted to the UL CDash.
The configuration of the script is managed by CMake.

[UL CDash](http://cdash.uni.lux/index.php?project=Modules-Check)

## Configure the scripts


Checkout the sources

    $> git clone git@github.com:ULHPC/modules-check.git modules-check-src


Create a directory to hold the configured scripts

    $> mkdir modules-check-MPI


Generate the script with CMake

    $> cd modules-check-MPI  
    $> cmake ../modules-check-src/MPI


## Testing an MPI stack


Running the tests in the current environment

    $> ctest



To run the tests for a given module

    $> module purge           
    $> module load OpenMPI    
    $> ctest


You can even try

    $> export PATH=/path/to/my/mpi/install:$PATH
    $> ctest



## Submitting test results


Using CTest, you can submit the results to the [UL CDash](http://cdash.uni.lux/index.php?project=Modules-Check) in the `Experimental` group:

    $> ctest -D Experimental


Alternatively, you can use the `test_and_submit.cmake` script to run and submit the results with explicit names.

    $> export CDASH_GROUP_NAME="Experimental"    
    $> export CDASH_SITE_NAME="Gaia cluster"    
    $> export CDASH_BUILD_NAME="MPI module 'OpenMPI'"  
    $> ctest -S test_and_submit.cmake



## Automated testing


The testing of all MPI modules can be triggered interactively with the `check_all_mpi_modules.sh`.

    $> ./check_all_mpi_modules.sh

This script lists all the available MPI module, tests all of them, and submit the results to the UL dashboard.
It must be run inside an OAR job.


The same script can be submitted to OAR to perform non-interactive testing of all MPI modules.

    $> oarsub -S ./check_all_mpi_modules.sh

This script already contains the required OAR parameters. It must be run on the cluster frontend.


## Adding new tests

The `CMakelists.txt` file describes the tests using the CMake/CTest language. 
It is simple enough so you can understand how to add a new test easily.

A few pointers to the CMake/CTest documentation:
* [CMake documentation](http://www.cmake.org/cmake/help/v2.8.10/cmake.html) 
* [CTest documentation](http://www.cmake.org/cmake/help/v2.8.10/ctest.html) 
* [add_test command](http://www.cmake.org/cmake/help/v2.8.10/cmake.html#command:add_test) 
* [Test properties](http://www.cmake.org/cmake/help/v2.8.10/cmake.html#section_PropertiesonTests) 

