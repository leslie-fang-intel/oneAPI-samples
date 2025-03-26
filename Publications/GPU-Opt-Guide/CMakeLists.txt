cmake_minimum_required(VERSION 3.21 FATAL_ERROR)
set(CMAKE_C_COMPILER icx)
set(CMAKE_CXX_COMPILER icpx)
set(_languages C CXX)

enable_testing()

project(GPUOptGuide
  LANGUAGES ${_languages}
  DESCRIPTION "Examples from oneAPI GPU Optimization Guide")

set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY BOTH)

find_package(IntelSYCL REQUIRED)

set(DPCPP_COMPILER ON)


string(CONCAT WARNING_CXX_FLAGS_STR
  "-Wall "
  "-Wextra "
  "-Winit-self "
  "-Wunused-function "
  "-Wuninitialized "
  "-Wmissing-declarations "
  "-fdiagnostics-color=auto "
)
string(REPLACE " " ";" COMMON_CXX_FLAGS "${WARNING_CXX_FLAGS_STR}")

function(add_example name)
  cmake_parse_arguments(FUNC_SRC "" "" "SOURCES" ${ARGN})
  if (FUNC_SRC_SOURCES)
    set(_src ${FUNC_SRC_SOURCES})
  else()
    set(_src ${name}.cpp)
  endif()
  add_executable(${name} ${_src})
  add_sycl_to_target(TARGET ${name} SOURCES ${_src})
  target_compile_options(${name} PRIVATE ${COMMON_CXX_FLAGS})
  target_link_options(${name} PRIVATE -fsycl-device-code-split=per_kernel)
  add_test(NAME ${name} COMMAND ${name} ${ARGN})
endfunction(add_example)

function(add_openmp_example name)
  set(_src ${name}.cpp)
  add_executable(${name} ${_src})
  add_sycl_to_target(TARGET ${name} SOURCES ${_src})
  target_compile_options(${name} PRIVATE ${COMMON_CXX_FLAGS})
  add_test(NAME ${name} COMMAND ${name} ${ARGN})
endfunction(add_openmp_example)

function(add_fortran_example name)
  if(CMAKE_Fortran_COMPILER)
    set(_src ${name}.f90)
    add_executable(${name} ${_src})
    add_sycl_to_target(TARGET ${name} SOURCES ${_src})
    target_compile_options(${name} PRIVATE -warn all)
    target_compile_options(${name} PRIVATE -fiopenmp -fopenmp-targets=spir64)
    set_target_properties(${name} PROPERTIES LINKER_LANGUAGE Fortran)
    target_link_options(${name} PRIVATE -fiopenmp -fopenmp-targets=spir64)
    add_test(NAME ${name} COMMAND ${name} ${ARGN})
  endif()
endfunction(add_fortran_example)

function(add_fixed_fortran_example name)
  if(CMAKE_Fortran_COMPILER)
    set(_src ${name}.f)
    add_executable(${name} ${_src})
    add_sycl_to_target(TARGET ${name} SOURCES ${_src})
    target_compile_options(${name} PRIVATE -warn all)
    target_compile_options(${name} PRIVATE -fiopenmp -fopenmp-targets=spir64)
    set_target_properties(${name} PROPERTIES LINKER_LANGUAGE Fortran)
    add_test(NAME ${name} COMMAND ${name} ${ARGN})
  endif()
endfunction(add_fixed_fortran_example)

function(add_mpi_example name)
  if(MPI_FOUND)
    set(_src ${name}.cpp)
    add_executable(${name} ${_src})
    add_sycl_to_target(TARGET ${name} SOURCES ${_src})
    target_compile_options(${name} PRIVATE -O3 -fiopenmp -fopenmp-targets=spir64)
    target_link_options(${name} PRIVATE -O3 -fiopenmp -fopenmp-targets=spir64)
    target_link_libraries(${name} PRIVATE MPI::MPI_CXX)
    add_test(NAME ${name} COMMAND ${name} ${ARGN})
  endif()
endfunction(add_mpi_example)


include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_subdirectory(work-group-size)
