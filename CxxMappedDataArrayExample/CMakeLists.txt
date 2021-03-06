cmake_minimum_required(VERSION 2.8.8)
project(CxxMappedDataArrayExample)

set(USE_CATALYST ON CACHE BOOL "Link the simulator with Catalyst")
if(USE_CATALYST)
  find_package(ParaView 4.1 REQUIRED COMPONENTS vtkPVPythonCatalyst)
  include("${PARAVIEW_USE_FILE}")
  set(Adaptor_SRCS
    FEAdaptor.cxx
    )
  add_library(CxxMappedDataArrayExampleAdaptor ${Adaptor_SRCS})
  target_link_libraries(CxxMappedDataArrayExampleAdaptor vtkPVPythonCatalyst vtkParallelMPI)
  add_definitions("-DUSE_CATALYST")
  if(NOT PARAVIEW_USE_MPI)
    message(SEND_ERROR "ParaView must be built with MPI enabled")
  endif()
else()
  find_package(MPI REQUIRED)
  include_directories(${MPI_C_INCLUDE_PATH})
endif()

add_executable(CxxMappedDataArrayExample FEDriver.cxx FEDataStructures.cxx)
if(USE_CATALYST)
  target_link_libraries(CxxMappedDataArrayExample LINK_PRIVATE CxxMappedDataArrayExampleAdaptor)
  include(vtkMPI)
  vtk_mpi_link(CxxMappedDataArrayExample)
else()
  target_link_libraries(CxxMappedDataArrayExample LINK_PRIVATE ${MPI_LIBRARIES})
endif()

option(BUILD_TESTING "Build Testing" OFF)
# Setup testing.
if (BUILD_TESTING)
  include(CTest)
  add_test(CxxMappedDataArrayExampleTest CxxMappedDataArrayExample ${CMAKE_CURRENT_SOURCE_DIR}/SampleScripts/feslicescript.py)
endif()
