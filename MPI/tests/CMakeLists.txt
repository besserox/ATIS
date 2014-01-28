cmake_minimum_required(VERSION 2.8.4)

project(ATIS-MPI-Tests)
include(CTest)
enable_testing()


# ------------------------- #
#  Find external programs   #
# ------------------------- #

# Look for numdiff command
find_program( NUMDIFF_COMMAND numdiff )

# Look for Rscript command
find_program( RSCRIPT_COMMAND Rscript )



# ------------------------- #
#  CMake options            #
# ------------------------- #

# Option for to select the Job Scheduler
set( JOB_SCHEDULER "None" CACHE STRING "Job Scheduler to use to find the machine file. Possible values are: None, OAR, PBS.")
set_property( CACHE JOB_SCHEDULER PROPERTY STRINGS "None" "OAR" "PBS" )

# Option for result comparison
set(COMPARE_RESULTS_DEFAULT_VALUE OFF)
if(NUMDIFF_COMMAND)
  set(COMPARE_RESULTS_DEFAULT_VALUE ON)
endif()
option( COMPARE_RESULTS "Compare the result of some tests with some reference results" "${COMPARE_RESULTS_DEFAULT_VALUE}" )

# Option for result plotting
set(PLOT_RESULTS_DEFAULT_VALUE OFF)
if(RSCRIPT_COMMAND)
  set(PLOT_RESULTS_DEFAULT_VALUE ON)
endif()
option( PLOT_RESULTS "Plot results of some tests" "${PLOT_RESULTS_DEFAULT_VALUE}" )


# ------------------------- #
#  Check CMake options      #
# ------------------------- #

# Result comparison requires the numdiff command
if( COMPARE_RESULTS AND NOT NUMDIFF_COMMAND )
  message( FATAL_ERROR "numdiff command not found! Cannot enable the result comparison" )
endif()

# Plotting results requires the Rscript command
if( PLOT_RESULTS AND NOT RSCRIPT_COMMAND )
  message( FATAL_ERROR "Rscript command not found! Cannot enable the result plotting" )
endif()



# ------------------------- #
#  Configure the tests      #
# ------------------------- #

set(OMB_NAME "osu-micro-benchmarks-3.9")
set(TEST_DIR "${CMAKE_BINARY_DIR}")
set(REFERENCE_DIR "${CMAKE_SOURCE_DIR}/reference_results")

# create MPI test scripts
configure_file( "${CMAKE_SOURCE_DIR}/compile-hello.sh.in"          "${TEST_DIR}/compile-hello.sh"                    @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/compile-omb.sh.in"            "${TEST_DIR}/compile-omb.sh"                      @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/get_machine_filename.sh.in"   "${TEST_DIR}/get_machine_filename.sh"             @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/mpirun.sh.in"                 "${TEST_DIR}/mpirun.sh"                           @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/mpirun_absolute_path.sh.in"   "${TEST_DIR}/mpirun_absolute_path.sh"             @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/mpirun_log_output.sh.in"      "${TEST_DIR}/mpirun_log_output.sh"                @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/plot_osu_bw.R.in"             "${TEST_DIR}/plot_osu_bw.R"                       @ONLY )




# ------------------------- #
#  List of actual tests     #
# ------------------------- #

# Failure pattern used to catch additional failures
set(FAILURE_PATTERN "(Usage|Error|error|Fail|FAIL|fail|Deprecated|not found)")

# Checking for mpicc 
add_test(10-MPI-which-mpicc       "which" "mpicc")
set_tests_properties(10-MPI-which-mpicc  PROPERTIES TIMEOUT 10)

# Checking for mpirun
add_test(20-MPI-which-mpirun      "which" "mpirun")
set_tests_properties(20-MPI-which-mpirun  PROPERTIES TIMEOUT 10)

# Compiling HelloWorld example
add_test(30-MPI-compilation-hello "${TEST_DIR}/compile-hello.sh")
set_tests_properties(30-MPI-compilation-hello   PROPERTIES TIMEOUT 10)

# Execution of HelloWorld with mpirun and the program as absolute path
add_test(40-MPI-mpirun_and_hello_with_absolute_path       "${TEST_DIR}/mpirun_absolute_path.sh" "${TEST_DIR}/hello")
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES DEPENDS 30-MPI-compilation-hello)
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES TIMEOUT 30)
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of HelloWorld with mpirun and the program as a relative path
add_test(50-MPI-mpirun_hello_as_relative_path             "${TEST_DIR}/mpirun_absolute_path.sh" "./hello")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES DEPENDS 30-MPI-compilation-hello)
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES TIMEOUT 30)
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES WORKING_DIRECTORY "${TEST_DIR}")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of HelloWorld with mpirun in the PATH 
add_test(60-MPI-mpirun_in_the_path_hello   "${TEST_DIR}/mpirun.sh" "${TEST_DIR}/hello")
set_tests_properties(60-MPI-mpirun_in_the_path_hello   PROPERTIES DEPENDS 30-MPI-compilation-hello)
set_tests_properties(60-MPI-mpirun_in_the_path_hello   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(60-MPI-mpirun_in_the_path_hello   PROPERTIES TIMEOUT 30)
set_tests_properties(60-MPI-mpirun_in_the_path_hello   PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(60-MPI-mpirun_in_the_path_hello   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Check for MPI libraries on remote node
add_test(70-MPI-mpirun_dynamic_libraries   "${TEST_DIR}/mpirun_absolute_path.sh" "ldd" "${TEST_DIR}/hello")
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES DEPENDS 30-MPI-compilation-hello)
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES TIMEOUT 30)
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Compiling OSU micro-benchmark
add_test(80-MPI-compilation-omb   "${TEST_DIR}/compile-omb.sh")
set_tests_properties(80-MPI-compilation-omb   PROPERTIES TIMEOUT 120)
set(OMB_DIR "${TEST_DIR}/${OMB_NAME}/mpi/pt2pt") 

# Execution of OMB osu_bw
set(OSU_BW_OUTPUT "${TEST_DIR}/osu_bw_output")
add_test(82-MPI-mpirun-osu_bw   "${TEST_DIR}/mpirun_log_output.sh" "${OSU_BW_OUTPUT}" "${OMB_DIR}/osu_bw")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES DEPENDS 80-MPI-compilation-omb)
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES REQUIRED_FILES "${OMB_DIR}/osu_bw")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES TIMEOUT 60)
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES PASS_REGULAR_EXPRESSION "OSU MPI Bandwidth Test")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of OMB osu_latency
set(OSU_LATENCY_OUTPUT "${TEST_DIR}/osu_latency_output")
add_test(84-MPI-mpirun-osu_latency   "${TEST_DIR}/mpirun_log_output.sh" "${OSU_LATENCY_OUTPUT}" "${OMB_DIR}/osu_latency")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES DEPENDS 80-MPI-compilation-omb)
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES REQUIRED_FILES "${OMB_DIR}/osu_latency")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES TIMEOUT 60)
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES PASS_REGULAR_EXPRESSION "OSU MPI Latency Test")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

if(COMPARE_RESULTS) 
  # Check the output of OMB osu_bw
  set(OSU_BW_REFERENCE "${REFERENCE_DIR}/osu_bw")
  add_test(82-MPI-mpirun-osu_bw_check   "${NUMDIFF_COMMAND}" "--overview" "--relative-tolerance=0.2" "--negative-differences" "${OSU_BW_REFERENCE}" "${OSU_BW_OUTPUT}")
  set_tests_properties(82-MPI-mpirun-osu_bw_check   PROPERTIES DEPENDS 82-MPI-mpirun-osu_bw)
  set_tests_properties(82-MPI-mpirun-osu_bw_check   PROPERTIES REQUIRED_FILES "${OSU_BW_REFERENCE};${OSU_BW_OUTPUT}")
  set_tests_properties(82-MPI-mpirun-osu_bw_check   PROPERTIES TIMEOUT 10)

  # Check the output of OMB osu_latency
  set(OSU_LATENCY_REFERENCE "${REFERENCE_DIR}/osu_latency")
  add_test(84-MPI-mpirun-osu_latency_check   "${NUMDIFF_COMMAND}" "--overview" "--relative-tolerance=0.2" "--positive-differences" "${OSU_LATENCY_REFERENCE}" "${OSU_LATENCY_OUTPUT}")
  set_tests_properties(84-MPI-mpirun-osu_latency_check   PROPERTIES DEPENDS 84-MPI-mpirun-osu_latency)
  set_tests_properties(84-MPI-mpirun-osu_latency_check   PROPERTIES REQUIRED_FILES "${OSU_LATENCY_REFERENCE};${OSU_LATENCY_OUTPUT}")
  set_tests_properties(84-MPI-mpirun-osu_latency_check   PROPERTIES TIMEOUT 10)
endif()

if(PLOT_RESULTS)
  # Plot the results of OMB osu_bw
  set(OSU_BW_PLOT "${OSU_BW_OUTPUT}.pdf")
  add_test(82-MPI-mpirun-osu_bw_plot "${TEST_DIR}/plot_osu_bw.R" "${OSU_BW_OUTPUT}")
  set_tests_properties(82-MPI-mpirun-osu_bw_plot   PROPERTIES DEPENDS 82-MPI-mpirun-osu_bw)
  set_tests_properties(82-MPI-mpirun-osu_bw_plot   PROPERTIES REQUIRED_FILES "${OSU_BW_OUTPUT}")
  set_tests_properties(82-MPI-mpirun-osu_bw_plot   PROPERTIES TIMEOUT 60)  
endif()
