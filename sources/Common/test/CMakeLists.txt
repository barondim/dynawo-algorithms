# Copyright (c) 2015-2021, RTE (http://www.rte-france.com)
# See AUTHORS.txt
# All rights reserved.
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, you can obtain one at http://mozilla.org/MPL/2.0/.
# SPDX-License-Identifier: MPL-2.0
#
# This file is part of Dynawo, an hybrid C++/Modelica open source suite
# of simulation tools for power systems.

set(MODULE_NAME dynawo_algorithms_Common_unittest)

set(MODULE_SOURCES
    TestBaseClasses.cpp
)

if (USE_MPI STREQUAL "YES")
  list(APPEND MODULE_SOURCES
        TestMPIContext.cpp)
endif()

add_executable(${MODULE_NAME} ${MODULE_SOURCES})

target_include_directories(${MODULE_NAME}
        PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../>
        )

target_link_libraries(${MODULE_NAME}
        dynawo_algorithms_Common
        dynawo_algorithms_Test
        Boost::system)

add_custom_target(${MODULE_NAME}-tests
  COMMAND ${CMAKE_COMMAND} -E env ${runtime_tests_ENV} $<TARGET_FILE:${MODULE_NAME}>
  DEPENDS
    ${MODULE_NAME}
  COMMENT "Running ${MODULE_NAME}...")

if(BUILD_TESTS_COVERAGE)
  set(EXTRACT_PATTERNS "'*/sources/Common/DYN*'")

  add_test_coverage(${MODULE_NAME}-tests "${EXTRACT_PATTERNS}")
endif()

if(BUILD_TESTS)
  add_test(${MODULE_NAME}-tests)
endif()
