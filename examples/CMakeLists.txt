#=============================================================================
#   CMake build system files
#
#   Copyright (c) 2014 pocl developers
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"), to deal
#   in the Software without restriction, including without limitation the rights
#   to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#   copies of the Software, and to permit persons to whom the Software is
#   furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included in
#   all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#   AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#   OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#   THE SOFTWARE.
#
#=============================================================================

# to install system packages required for all examples:
#
# on Archlinux:
#    pacman -S blas cblas lapack lapacke openmpi svn  freeimage  python-numpy python2-numpy  python-mako python2-mako
#
# on Ubuntu:
#    apt-get install libblas-dev liblapack-dev  liblapacke-dev  libopenmpi-dev   subversion python-numpy python-mako
#

add_subdirectory("example1")
add_subdirectory("example1-spir32")
add_subdirectory("example1-spir64")
add_subdirectory("example2")
add_subdirectory("example2a")
add_subdirectory("poclcc")
add_subdirectory("scalarwave")
add_subdirectory("trig")
add_subdirectory("EinsteinToolkit")

# TODO:   opencl-book-samples
set(ALL_TESTSUITES
    AMD AMDSDK2.9 AMDSDK3.0
    ASL arrayfire clBLAS clFFT
    CloverLeaf Halide IntelSVM
    opencl-book-samples OpenCV
    Parboil piglit PyOpenCL
    Rodinia VexCL ViennaCL)

if(NOT DEFINED ENABLE_TESTSUITES)
  set(ENABLE_TESTSUITES "all")
endif()

if("${ENABLE_TESTSUITES}" STREQUAL "all")
  set(ENABLE_TESTSUITES ${ALL_TESTSUITES})
endif()

include(ExternalProject)

# invoke this to build all examples
add_custom_target(prepare_examples)

if(ENABLE_TESTSUITES AND (NOT DEFINED ACTUALLY_ENABLED_TESTSUITES))

    unset(CMAKE_MODULE_PATH) # Use CMake builtin find module
    find_package(Git QUIET)
    set_expr(HAVE_GIT GIT_EXECUTABLE)
    find_package(Subversion QUIET)
    set_expr(HAVE_SVN Subversion_SVN_EXECUTABLE)

  message(STATUS "Trying to enable testsuites: ${ENABLE_TESTSUITES}")

  set(ACTUALLY_ENABLED_TESTSUITES "")

  if(NOT DEFINED TESTSUITE_BASEDIR)
    # TODO maybe current src dir ?
    set(TESTSUITE_BASEDIR "${CMAKE_CURRENT_BINARY_DIR}")
  endif()
  if(NOT IS_DIRECTORY "${TESTSUITE_BASEDIR}")
    message(FATAL_ERROR "TESTSUITE_BASEDIR (${TESTSUITE_BASEDIR}) is not a directory.")
  endif()

  if(NOT DEFINED TESTSUITE_SOURCE_BASEDIR)
    set(TESTSUITE_SOURCE_BASEDIR "${CMAKE_CURRENT_SOURCE_DIR}")
  endif()
  if(NOT IS_DIRECTORY "${TESTSUITE_SOURCE_BASEDIR}")
    message(FATAL_ERROR "TESTSUITE_SOURCE_BASEDIR (${TESTSUITE_SOURCE_BASEDIR}) is not a directory.")
  endif()

  message(STATUS "Testsuite base dirs: ")
  message(STATUS "    binary: ${TESTSUITE_BASEDIR}")
  message(STATUS "    source: ${TESTSUITE_SOURCE_BASEDIR}")

  foreach(TESTSUITE IN LISTS ENABLE_TESTSUITES)
    list(FIND ALL_TESTSUITES ${TESTSUITE} INDEX)
    if(${INDEX} GREATER -1)
      if(IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/${TESTSUITE}")
        add_subdirectory("${TESTSUITE}")
      else()
        message(FATAL_ERROR "Cannot find source dir for testsuite: ${TESTSUITE}")
      endif()
    else()
      message(WARNING "Unknown testsuite ${TESTSUITE} requested")
    endif()
  endforeach()

  set(ACTUALLY_ENABLED_TESTSUITES "${ACTUALLY_ENABLED_TESTSUITES}" CACHE STRING "Actually available & enabled testsuites")
endif()

set(ACTUALLY_ENABLED_TESTSUITES ${ACTUALLY_ENABLED_TESTSUITES} PARENT_SCOPE)
set(ALL_TESTSUITES ${ALL_TESTSUITES} PARENT_SCOPE)

set(DISABLED "${ALL_TESTSUITES}")
foreach(TS ${ACTUALLY_ENABLED_TESTSUITES})
  list(REMOVE_ITEM DISABLED ${TS})
endforeach()

set(DISABLED_TESTSUITES ${DISABLED} PARENT_SCOPE)

######################################################################################

if(ACTUALLY_ENABLED_TESTSUITES MATCHES "AMD")

  if(NOT MSVC)
    pkg_check_modules(GLEW glew)
  endif()

  if(GLEW_FOUND)
    set(HAVE_GLEW 1 PARENT_SCOPE)
  else()
    set(HAVE_GLEW 0 PARENT_SCOPE)
    message(WARNING "libGLEW not found. A few tests from AMD testsuite will not work")
  endif()

endif()
