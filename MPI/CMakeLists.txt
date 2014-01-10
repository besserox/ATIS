cmake_minimum_required(VERSION 2.8.4)

project(ATIS-MPI)



# ------------------------- #
#  CMake options            #
# ------------------------- #

# Option to force a given MODULE_PATH
SET( MODULE_PATH "default" CACHE STRING "Force a MODULE_PATH before running the tests. ('default' means no override)")

# Option to give a SITE_NAME
SET( CDASH_SITE_NAME "default" CACHE STRING "Force a SITE_NAME for CDash submission. ('default' means a SITE_NAME extracted from the hostname)")

# Option for to select the Job Scheduler
SET( JOB_SCHEDULER "None" CACHE STRING "Job Scheduler to use to find the machine file. Possible values are: None, OAR, PBS.")
SET_PROPERTY( CACHE JOB_SCHEDULER PROPERTY STRINGS "None" "OAR" "PBS" )



# ------------------------- #
#  Configure the tests      #
# ------------------------- #

# create management scripts
configure_file( "${CMAKE_SOURCE_DIR}/run_tests.sh.in"                    "${CMAKE_BINARY_DIR}/run_tests.sh"                @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/test_and_submit.cmake.in"           "${CMAKE_BINARY_DIR}/test_and_submit.cmake"       @ONLY )
#configure_file( "${CMAKE_SOURCE_DIR}/check_all_mpi_modules.sh.in"        "${CMAKE_BINARY_DIR}/check_all_mpi_modules.sh"    @ONLY )
#configure_file( "${CMAKE_SOURCE_DIR}/update_mpi_module_list.sh.in"       "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh"   @ONLY )

# generate initial module list
#execute_process( COMMAND "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh" )

# create log directory
file( MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/logs" )
