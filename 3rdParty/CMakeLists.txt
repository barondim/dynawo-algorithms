# Copyright (c) 2022, RTE (http://www.rte-france.com)
# See AUTHORS.txt
# All rights reserved.
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, you can obtain one at http://mozilla.org/MPL/2.0/.
# SPDX-License-Identifier: MPL-2.0
#
# This file is part of Dynawo, an hybrid C++/Modelica open source suite
# of simulation tools for power systems.

# Minimum required (for ExternalProject_Add)
cmake_minimum_required(VERSION 3.9.6 FATAL_ERROR)

project("dynawo-algorithms-3rd"
LANGUAGES C CXX
)

get_filename_component(DYNAWO_HOME ${DYNAWO_HOME} ABSOLUTE)

include(ExternalProject)
include(${DYNAWO_HOME}/share/cmake/CPUCount.cmake)
if(NOT DEFINED CPU_COUNT)
  message(FATAL_ERROR "CPUCount.cmake: file not found.")
endif()

set(DOWNLOAD_DIR ${CMAKE_INSTALL_PREFIX}/src CACHE PATH "Directory where 3rd parties are downloaded.")
set(TMP_DIR ${CMAKE_INSTALL_PREFIX}/tmp CACHE PATH "Directory where 3rd parties temporary files are created.")

if(UNIX)
  set(gperftools_version "2.6.1")
  if(DEFINED ENV{DYNAWO_ALGORITHMS_GPERFTOOLS_DOWNLOAD_URL})
    set(gperftools_prefix_url $ENV{DYNAWO_ALGORITHMS_GPERFTOOLS_DOWNLOAD_URL})
  else()
    set(gperftools_prefix_url https://github.com/gperftools/gperftools/releases/download)
  endif()
  set(gperftools_url ${gperftools_prefix_url}/gperftools-${gperftools_version}/gperftools-${gperftools_version}.tar.gz)
  set(gperftools_md5 e010d656b8b62e61c0af0c4d79843782)
  find_library(tcmalloc NAMES "tcmalloc" HINTS ${CMAKE_INSTALL_PREFIX}/gperftools/lib)
  if (NOT tcmalloc)
    ExternalProject_Add(gperftools
            INSTALL_DIR         ${CMAKE_INSTALL_PREFIX}/gperftools
            DOWNLOAD_DIR        ${CMAKE_CURRENT_SOURCE_DIR}/gperftools
            TMP_DIR             ${TMP_DIR}
            STAMP_DIR           ${DOWNLOAD_DIR}/gperftools-stamp
            SOURCE_DIR          ${DOWNLOAD_DIR}/gperftools
            URL                 ${gperftools_url}
            URL_MD5             ${gperftools_md5}
            BUILD_IN_SOURCE     1

            CONFIGURE_COMMAND   <SOURCE_DIR>/configure
                                "CC=${CMAKE_C_COMPILER}"
                                "CXX=${CMAKE_CXX_COMPILER}"
                                "CXXFLAGS=-fno-strict-aliasing -DTCMALLOC_LARGE_PAGES -O3"
                                "CFLAGS=-fno-strict-aliasing -DTCMALLOC_LARGE_PAGES -O3"
                                --prefix=<INSTALL_DIR>
                                --disable-static
                                --disable-libunwind
            BUILD_COMMAND       make -j ${CPU_COUNT}
    )
  endif()
endif()

if(USE_MPI STREQUAL "YES")
  set(mpich_version "3.4.2")
  if(DEFINED ENV{DYNAWO_ALGORITHMS_MPICH_DOWNLOAD_URL})
    set(mpich_prefix_url $ENV{DYNAWO_ALGORITHMS_MPICH_DOWNLOAD_URL})
  else()
    set(mpich_prefix_url http://www.mpich.org/static/downloads)
  endif()
  set(mpich_url ${mpich_prefix_url}/${mpich_version}/mpich-${mpich_version}.tar.gz)
  set(mpich_md5 6ee1cfff98728e5160c6e78bdb1986ca)

  find_package(MPI QUIET)
  if(MPI_FOUND)
    add_custom_target(mpich)
  else()
    ExternalProject_Add(mpich
        INSTALL_DIR         ${CMAKE_INSTALL_PREFIX}/mpich
        DOWNLOAD_DIR        ${CMAKE_CURRENT_SOURCE_DIR}/mpich
        TMP_DIR             ${TMP_DIR}
        STAMP_DIR           ${DOWNLOAD_DIR}/mpich-stamp
        SOURCE_DIR          ${DOWNLOAD_DIR}/mpich
        URL                 ${mpich_url}
        URL_MD5             ${mpich_md5}
        BUILD_IN_SOURCE     1

        CONFIGURE_COMMAND   <SOURCE_DIR>/configure
                            --prefix=<INSTALL_DIR>
                            "CC=${CMAKE_C_COMPILER}"
                            "CXX=${CMAKE_CXX_COMPILER}"
                            "CFLAGS=-m64 -O3 -DNDEBUG"
                            "CXXFLAGS=-m64 -O3 -DNDEBUG"
                            "FCFLAGS=-m64 -O3 -DNDEBUG"
                            "LDFLAGS=-m64 -O3 -DNDEBUG"
                            --with-device=ch3:nemesis
                            --disable-fortran
                            --enable-fast=all,O3,ndebug
        BUILD_COMMAND       make -j ${CPU_COUNT}
    )
    ExternalProject_Get_Property(mpich install_dir)
    set(MPI_HOME ${install_dir})
  endif()
endif()
