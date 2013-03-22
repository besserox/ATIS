cmake_minimum_required(VERSION 2.8.4)

project(ModulesCheckMPI)
include(CTest)
enable_testing()

set(OMB_NAME "osu-micro-benchmarks-3.9")


# ------------------------- #
#  Configure the tests      #
# ------------------------- #

set(TEST_DIR "${CMAKE_BINARY_DIR}/tests")

# create MPI test scripts
configure_file( "${CMAKE_SOURCE_DIR}/tests/compile-hello.sh.in"          "${TEST_DIR}/compile-hello.sh"                    @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/tests/compile-omb.sh.in"            "${TEST_DIR}/compile-omb.sh"                      @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/tests/mpirun.sh.in"                 "${TEST_DIR}/mpirun.sh"                           @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/tests/mpirun_absolute_path.sh.in"   "${TEST_DIR}/mpirun_absolute_path.sh"             @ONLY )

# create management scripts
configure_file( "${CMAKE_SOURCE_DIR}/test_and_submit.cmake.in"           "${CMAKE_BINARY_DIR}/test_and_submit.cmake"       @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/check_all_mpi_modules.sh.in"        "${CMAKE_BINARY_DIR}/check_all_mpi_modules.sh"    @ONLY )
configure_file( "${CMAKE_SOURCE_DIR}/update_mpi_module_list.sh.in"       "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh"   @ONLY )

# generate initial module list
execute_process( COMMAND "${CMAKE_BINARY_DIR}/update_mpi_module_list.sh" )

# create log directory
file( MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/logs" )



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
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES DEPENDS MPI-compilation-hello)
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES TIMEOUT 30)
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(40-MPI-mpirun_and_hello_with_absolute_path  PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of HelloWorld with mpirun and the program as a relative path
add_test(50-MPI-mpirun_hello_as_relative_path             "${TEST_DIR}/mpirun_absolute_path.sh" "./hello")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES DEPENDS MPI-compilation-hello)
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES TIMEOUT 30)
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES WORKING_DIRECTORY "${TEST_DIR}")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(50-MPI-mpirun_hello_as_relative_path   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of HelloWorld with mpirun in the PATH 
add_test(60-MPI-mpirun_in_the_path   "${TEST_DIR}/mpirun.sh" "${TEST_DIR}/hello")
set_tests_properties(60-MPI-mpirun_in_the_path   PROPERTIES DEPENDS MPI-compilation-hello)
set_tests_properties(60-MPI-mpirun_in_the_path   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(60-MPI-mpirun_in_the_path   PROPERTIES TIMEOUT 30)
set_tests_properties(60-MPI-mpirun_in_the_path   PROPERTIES PASS_REGULAR_EXPRESSION "Hello, World, I am")
set_tests_properties(60-MPI-mpirun_in_the_path   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Check for MPI libraries on remote node
add_test(70-MPI-mpirun_dynamic_libraries   "${TEST_DIR}/mpirun_absolute_path.sh" "ldd" "${TEST_DIR}/hello")
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES DEPENDS MPI-compilation-hello)
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES REQUIRED_FILES "${TEST_DIR}/hello")
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES TIMEOUT 30)
set_tests_properties(70-MPI-mpirun_dynamic_libraries   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Compiling OSU micro-benchmark
add_test(80-MPI-compilation-omb   "${TEST_DIR}/compile-omb.sh")
set_tests_properties(80-MPI-compilation-omb   PROPERTIES TIMEOUT 120)
set(OMB_DIR "${TEST_DIR}/${OMB_NAME}/mpi/pt2pt") 

# Execution of OMB osu_bw
add_test(82-MPI-mpirun-osu_bw   "${TEST_DIR}/mpirun_absolute_path.sh" "${OMB_DIR}/osu_bw")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES DEPENDS MPI-compilation-omb)
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES REQUIRED_FILES "${OMB_DIR}/osu_bw")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES TIMEOUT 60)
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES PASS_REGULAR_EXPRESSION "OSU MPI Bandwidth Test")
set_tests_properties(82-MPI-mpirun-osu_bw   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

# Execution of OMB osu_latency
add_test(84-MPI-mpirun-osu_latency   "${TEST_DIR}/mpirun_absolute_path.sh" "${OMB_DIR}/osu_latency")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES DEPENDS MPI-compilation-omb)
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES REQUIRED_FILES "${OMB_DIR}/osu_latency")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES TIMEOUT 60)
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES PASS_REGULAR_EXPRESSION "OSU MPI Latency Test")
set_tests_properties(84-MPI-mpirun-osu_latency   PROPERTIES FAIL_REGULAR_EXPRESSION "${FAILURE_PATTERN}")

