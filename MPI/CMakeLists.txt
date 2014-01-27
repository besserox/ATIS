cmake_minimum_required(VERSION 2.8.4)

project(ATIS-MPI)



# ------------------------- #
#  Find external programs   #
# ------------------------- #

# Look for numdiff command
find_program( NUMDIFF_COMMAND numdiff )



# ------------------------- #
#  CMake options            #
# ------------------------- #

# Option to force a given MODULE_PATH
set( CUSTOM_MODULE_PATH "" CACHE STRING "Force a MODULE_PATH before running the tests. ('default' means no override)")

# Option to give a SITE_NAME
set( CDASH_SITE_NAME "default" CACHE STRING "Force a SITE_NAME for CDash submission. ('default' means a SITE_NAME extracted from the hostname)")

# Option for to select the Job Scheduler
set( JOB_SCHEDULER "None" CACHE STRING "Job Scheduler to use to find the machine file. Possible values are: None, OAR, PBS.")
set_property( CACHE JOB_SCHEDULER PROPERTY STRINGS "None" "OAR" "PBS" )

# Option to force a given MODULE_PATH
set( CUSTOM_MODULE_PATH "" CACHE STRING "Force a MODULE_PATH before running the tests. ('default' means no override)")

# Option for result comparison
set(RESULT_COMPARISON_DEFAULT_VALUE OFF)
if(NUMDIFF_COMMAND)
  set(RESULT_COMPARISON_DEFAULT_VALUE ON)
endif()
option( RESULT_COMPARISON "Compare the result of some tests with some reference results" "${RESULT_COMPARISON_DEFAULT_VALUE}" )



# ------------------------- #
#  Check CMake options      #
# ------------------------- #

# Result comparison requires the numdiff command
if( RESULT_COMPARISON AND NOT NUMDIFF_COMMAND )
  message( FATAL_ERROR "numdiff command not found! Cannot enable the result comparison" )
endif()





# ------------------------- #
#  Configure the tests      #
# ------------------------- #

# create management scripts
configure_file( "${CMAKE_SOURCE_DIR}/run_tests.sh.in"                                "${CMAKE_BINARY_DIR}/run_tests.sh"                          @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/test_and_submit.cmake.in"                       "${CMAKE_BINARY_DIR}/test_and_submit.cmake"                 @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/update_mpi_module_list.sh.in"                   "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh"             @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/oar_nightly_check_all_mpi_modules.sh.in"        "${CMAKE_BINARY_DIR}/oar_nightly_check_all_mpi_modules.sh"    @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/nightly_check_all_mpi_modules.sh.in"            "${CMAKE_BINARY_DIR}/nightly_check_all_mpi_modules.sh"        @ONLY )

# generate initial module list
execute_process( COMMAND "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh" )

# create log directory
file( MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/logs" )
