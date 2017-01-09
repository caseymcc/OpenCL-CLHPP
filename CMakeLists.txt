cmake_minimum_required(VERSION 3.0)

include(HunterGate.cmake)

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.17.12.tar.gz"
    SHA1 "74cfa490ef0d3ff8916523bbf5add53049aad7be"
)


project(CLCPP)
enable_testing()

option(BUILD_DOCS "Build Documentation" ON)
option(BUILD_EXAMPLES "Build Examples" ON)
option(BUILD_TESTS "Build Unit Tests" ON)
option(OPENCL_CLHPP_VERSION2_ONLY "Gernerate only cl2.hpp" OFF)

if(DEFINED ENV{AMDAPPSDKROOT})
  set(OPENCL_DIST_DIR $ENV{AMDAPPSDKROOT} CACHE PATH "OpenCL source dir")
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR ${OPENCL_DIST_DIR}/lib/x86_64 )
  else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR ${OPENCL_DIST_DIR}/lib/x86 )
  endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )
elseif(DEFINED ENV{INTELOCLSDKROOT})
  set(OPENCL_DIST_DIR $ENV{INTELOCLSDKROOT} CACHE PATH "OpenCL source dir")
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR ${OPENCL_DIST_DIR}/lib/x64 )
  else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR ${OPENCL_DIST_DIR}/lib/x86 )
  endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )
else()
  set(OPENCL_DIST_DIR "Hello" CACHE PATH "OpenCL source dir")
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR "" )
  else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set(OPENCL_LIB_DIR "" )
  endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )  
endif()

if( CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang" )
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pthread")
endif()

set(OPENCL_INCLUDE_DIR ${OPENCL_DIST_DIR}/include CACHE PATH "OpenCL source dir")
set(UNITY_DIR $ENV{UNITY_DIR} CACHE PATH "Unity dir")
set(CMOCK_DIR $ENV{CMOCK_DIR} CACHE PATH "CMock dir")

if(BUILD_DOCS)
add_subdirectory(docs)
endif(BUILD_DOCS)

add_subdirectory(include)

if(BUILD_EXAMPLES)
add_subdirectory(examples)
endif(BUILD_EXAMPLES)

if(BUILD_TESTS)
add_subdirectory(tests)
endif(BUILD_TESTS)
