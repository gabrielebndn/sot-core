# Copyright 2010, François Bleibel, Olivier Stasse, JRL, CNRS/AIST
#
# This file is part of sot-core.
# sot-core is free software: you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of
# the License, or (at your option) any later version.
#
# sot-core is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Lesser Public License for more details.  You should have
# received a copy of the GNU Lesser General Public License along with
# sot-core. If not, see <http://www.gnu.org/licenses/>.

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

INCLUDE(cmake/base.cmake)
INCLUDE(cmake/boost.cmake)
INCLUDE(cmake/eigen.cmake)
INCLUDE(cmake/python.cmake)

SET(PROJECT_NAMESPACE stack-of-tasks)
SET(PROJECT_NAME sot-core)
SET(PROJECT_DESCRIPTION "Hierarchical task solver plug-in for dynamic-graph.")
SET(PROJECT_URL "http://github.com/${PROJECT_NAMESPACE}/${PROJECT_NAME}")

SET(DOXYGEN_USE_MATHJAX YES)

# Disable -Werror on Unix for now.
SET(CXX_DISABLE_WERROR True)

SET(PKG_CONFIG_ADDITIONAL_VARIABLES
  ${PKG_CONFIG_ADDITIONAL_VARIABLES}
  plugindirname
  plugindir
  )

SETUP_PROJECT()

PKG_CONFIG_APPEND_LIBS("sot-core")

# Search for dependencies.
# Boost
SET(BOOST_COMPONENTS thread filesystem program_options unit_test_framework system regex )
SEARCH_FOR_EIGEN()

ADD_REQUIRED_DEPENDENCY("dynamic-graph >= 3.0.0")

OPTION (BUILD_PYTHON_INTERFACE "Build the python binding" ON)
OPTION (INSTALL_PYTHON_INTERFACE_ONLY "Install *ONLY* the python binding" OFF)

IF(BUILD_PYTHON_INTERFACE)
  FINDPYTHON(2.7 EXACT REQUIRED)
  STRING(REGEX REPLACE "-" "_" PY_NAME ${PROJECT_NAME})
  SET(${PY_NAME}_INSTALL_DIR ${PYTHON_SITELIB}/${PY_NAME})
  INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_DIRS})
  SET(PYTHON_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/${PYTHON_SITELIB}/dynamic_graph/sot/core)
  ADD_REQUIRED_DEPENDENCY("dynamic-graph-python >= 3.0.0")

ENDIF(BUILD_PYTHON_INTERFACE)
ADD_COMPILE_DEPENDENCY ("pinocchio >= 2.0.0")

SEARCH_FOR_BOOST()

ADD_SUBDIRECTORY(include)
ADD_SUBDIRECTORY(src)
ADD_SUBDIRECTORY(unitTesting)
IF (NOT INSTALL_PYTHON_INTERFACE_ONLY)
  ADD_SUBDIRECTORY(doc)
ENDIF(NOT INSTALL_PYTHON_INTERFACE_ONLY)

# **********************************
# Robot_utils_sot_py PYTHON module *
# **********************************
IF(BUILD_PYTHON_INTERFACE)
  PYTHON_ADD_MODULE(robot_utils_sot_py src/tools/robot-utils-py.cpp)
  PKG_CONFIG_USE_DEPENDENCY(robot_utils_sot_py dynamic-graph)
  PKG_CONFIG_USE_DEPENDENCY(robot_utils_sot_py pinocchio)
  TARGET_LINK_LIBRARIES(robot_utils_sot_py ${Boost_LIBRARIES} ${PYTHON_LIBRARIES} ${LIBRARY_NAME})
  TARGET_LINK_BOOST_PYTHON(robot_utils_sot_py)
  INSTALL(TARGETS robot_utils_sot_py DESTINATION ${PYTHON_INSTALL_DIR})
ENDIF(BUILD_PYTHON_INTERFACE)

# We do not want the project to be finalized if this is
# to install only the  python interface.
IF (NOT INSTALL_PYTHON_INTERFACE_ONLY)
  SETUP_PROJECT_FINALIZE()
ENDIF(NOT INSTALL_PYTHON_INTERFACE_ONLY)
