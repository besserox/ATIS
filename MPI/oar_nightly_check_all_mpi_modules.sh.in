#!/bin/bash -l
#OAR -l /nodes=2
#OAR -n ATIS-MPI
#OAR -O @CMAKE_BINARY_DIR@/logs/check_all_mpi_modules.%jobid%.log
#OAR -E @CMAKE_BINARY_DIR@/logs/check_all_mpi_modules.%jobid%.log

"@CMAKE_BINARY_DIR@/nightly_check_all_mpi_modules.sh"
