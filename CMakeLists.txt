#
# Copyright 2016 National Renewable Energy Laboratory
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

cmake_minimum_required(VERSION 2.8.12)
project(OpenFAST Fortran C)

include(${CMAKE_SOURCE_DIR}/cmake/FastCmakeUtils.cmake)
include(${CMAKE_SOURCE_DIR}/cmake/FastFortranOptions.cmake)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# CMake Configuration variables
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING
    "Choose the build type: Debug Release" FORCE)
endif (NOT CMAKE_BUILD_TYPE)

option(BUILD_SHARED_LIBS "Enable building shared libraries" off)
option(DOUBLE_PRECISION "Treat REAL as double precision" on)
option(USE_DLL_INTERFACE "Enable runtime loading of dynamic libraries" on)
option(FPE_TRAP_ENABLED "Enable FPE trap in compiler options" off)
option(ORCA_DLL_LOAD "Enable OrcaFlex Library Load" off)

# Setup Fortran Compiler options based on architecture/compiler
set_fast_fortran()

if (USE_DLL_INTERFACE)
  add_definitions(-DUSE_DLL_INTERFACE)
endif (USE_DLL_INTERFACE)

if (FPE_TRAP_ENABLED)
  add_definitions(-DFPE_TRAP_ENABLED)
endif (FPE_TRAP_ENABLED)

# Setup dependencies
find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)

########################################################################
# Build rules for FAST Registry
#
add_subdirectory(modules-local/fast-registry)


########################################################################
# FAST Core modules
#
set(FAST_MODULES_LOCAL
  nwtc-library
  inflowwind
  aerodyn
  aerodyn14
  servodyn
  elastodyn
  beamdyn
  subdyn
  hydrodyn
  orcaflex-interface
  extptfm
  openfoam
  fast-library
  )

########################################################################
# FAST External modules
#
set(FAST_MODULES_EXTERNAL
  feamooring
  moordyn
  icedyn
  icefloe
  map
  )

set(FAST_REGISTRY_INCLUDES "" CACHE INTERNAL "Registry includes paths")
set_registry_includes("modules-local" ${FAST_MODULES_LOCAL})
set_registry_includes("modules-ext" ${FAST_MODULES_EXTERNAL})
# Fix non-standard path addition to FAST_REGISTRY_INCLUDES in icefloe module
set(FAST_REGISTRY_INCLUDES
  ${FAST_REGISTRY_INCLUDES} -I ${CMAKE_SOURCE_DIR}/modules-ext/icefloe/src/interfaces/FAST/
  CACHE INTERNAL "Registry includes paths")

foreach(IDIR IN ITEMS ${FAST_MODULES_LOCAL})
  add_subdirectory("${CMAKE_SOURCE_DIR}/modules-local/${IDIR}")
endforeach(IDIR IN ITEMS ${FAST_MODULES_LOCAL})

foreach(IDIR IN ITEMS ${FAST_MODULES_EXTERNAL})
  add_subdirectory("${CMAKE_SOURCE_DIR}/modules-ext/${IDIR}")
endforeach(IDIR IN ITEMS ${FAST_MODULES_EXTERNAL})

option(BUILD_DOCUMENTATION "Build documentation." OFF)
if(BUILD_DOCUMENTATION)
   add_subdirectory(docs)
endif()

add_subdirectory(glue-codes)
