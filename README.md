# Automatic Testing of Installed Software

ATIS is not developed or supported.

For similar better projects, have a look at:
- https://github.com/HPC-buildtest/buildtest-framework
- https://github.com/eth-cscs/reframe


## Description

Automatic Testing of Installed Software is a testing framework to validate the various flavors of software installed on an HPC site. It is composed of a set of unit tests, a runtime and a result-gathering dashboard. These tests are user-oriented as they assess the basic features that a general user expect to work on an HPC platform.

Currently, it only focuses on generic MPI functionality as it is one complex and critical component of an HPC platform, but it will be extended to compilers, libraries and performance validation and regression in the future.

HPC centers tend to provide a wide choice a software. Different users requires different software, but also different versions of the same software. Combined with the different compilers, MPI stacks, library dependencies, there is an explosion of software flavors installed on an HPC site.

## Usage

Check usage example in the [MPI sub-directory](MPI).

