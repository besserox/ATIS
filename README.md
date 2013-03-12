-*- mode: markdown; mode: auto-fill; fill-column: 80 -*-
`README` -- [HPC @ UL](http://hpc.uni.lu)

        Time-stamp: <Mar 2013-03-12 11:44 svarrette>

-------------------

# UL HPC Launcher scripts

## Synopsis

This repository holds a CTest-based framework for the automatic testing of
generated modules for the [UL HPC](https://hpc.uni.lu) platform. 

# Contributing to this repository 

## Pre-requisites

### Git

You should become familiar (if not yet) with Git. Consider these resources:

* [Git book](http://book.git-scm.com/index.html)
* [Github:help](http://help.github.com/mac-set-up-git/)
* [Git reference](http://gitref.org/)

### git-flow

The Git branching model for this repository follows the guidelines of [gitflow](http://nvie.com/posts/a-successful-git-branching-model/).
In particular, the central repo (on `github.com`) holds two main branches with an infinite lifetime:

* `production`: the *production-ready* benchmark data 
* `devel`: the main branch where the latest developments interviene. This is
  the *default* branch you get when you clone the repo.

### Local repository setup

This repository is hosted on out [GitHub](https://github.com/ULHPC/modules-check).
Once cloned, initiate the potential git submodules etc. by running: 

    $> cd modules-check 
    $> make setup
