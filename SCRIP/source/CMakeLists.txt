set (LIB_NAME SCRIP)

find_package(OpenMP REQUIRED)

#find_package(MPI)
#if (${MPI_Fortran_FOUND})
#	file(GLOB sources ./*.F90 ./*.f mpi/*.F90)
#	add_definitions("-DMPI")
#else()
	file(GLOB sources ./*.F90 ./*.f serial/*.F90)
#endif()
add_library ( ${LIB_NAME} ${sources})
target_include_directories(${LIB_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# ifort and gfortran need the flag -cpp to interpret definitions like -DMRM2MHM
# the nag compiler is not able to interpret the flag -cpp but can interpret these definitions anyway
# so we check whether the compiler is able to use the flag -cpp
# for that we need the module CheckFortranCompilerFlag
include(CheckFortranCompilerFlag)
CHECK_Fortran_COMPILER_FLAG("-cpp" CPP_FLAG)
# if the flag exists, we add it to the compilation flags
if (CPP_FLAG)
	set(ADDITIONAL_GCC_FLAGS "-cpp")
endif()

# set specific place where to search for the netCDF directory
set(CMAKE_NETCDF_DIR " " CACHE STRING "set set specific place where to search for the netCDF directory")
message(STATUS "search in additional directory ${CMAKE_NETCDF_DIR} for netCDF")

# additional cmake-modules created for the purpose of finding netCDF or other libraries ly in the source_directory in
# a folder named cmake-modules. This command tells cmake to search there for Find<module>.cmake files
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../../cmake-modules)
set (NETCDF_F90 "YES")
# the FindNetCDFF.cmake file can be found after we added the cmake-modules folder to the CMAKE_MODULE_PATH
# the build fails, if it is not present
find_package(NetCDFF REQUIRED)
# from that module we gain the following variables:
# NETCDF_INCLUDES	: the include directory
# NETCDF_LINK_LIBRARIES : the absolute path to and with the libraries
# NETCDF_CFLAGS_OTHER	: additional compilation flags
# NETCDF_LDFLAGS_OTHER	: additional linking flags

# set compiling flags for debug and relese version
if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU")
    # set the preprocessor
    target_compile_definitions(${LIB_NAME} PRIVATE "-DGFORTRAN")
endif()
if(CMAKE_Fortran_COMPILER_ID MATCHES "PGI")
    # set the preprocessor
    target_compile_definitions(${LIB_NAME} PRIVATE "-DpgiFortran")
endif()
if(CMAKE_ABSOFT)
    # set the preprocessor
    target_compile_definitions(${LIB_NAME} PRIVATE "-DABSOFT")
endif()
if(CMAKE_Fortran_COMPILER_ID MATCHES "NAG")
	target_compile_options(${LIB_NAME} PUBLIC  "-kind=byte")
endif()

set_target_properties ( ${LIB_NAME}
  PROPERTIES
  COMPILE_FLAGS "${NETCDF_CFLAGS_OTHER} ${OpenMP_Fortran_FLAGS} ${MPI_Fortran_COMPILE_FLAGS} ${ADDITIONAL_GCC_FLAGS}"
  LINK_FLAGS "${NETCDF_LDFLAGS_OTHER} ${OpenMP_Fortran_FLAGS} ${MPI_Fortran_LINK_FLAGS}"
)

target_link_libraries(${LIB_NAME} ${NETCDF_LINK_LIBRARIES} ${OpenMP_Fortran_LIBRARIES} ${MPI_Fortran_LIBRARIES} )
target_include_directories(${LIB_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR} ${NETCDF_INCLUDES} ${OpenMP_Fortran_LIBRARY} ${MPI_Fortran_INCLUDE_PATH})
