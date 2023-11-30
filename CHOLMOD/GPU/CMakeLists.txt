#-------------------------------------------------------------------------------
# SuiteSparse/CHOLMOD/GPU/CMakeLists.txt:  cmake for CHOLMOD/GPU module
#-------------------------------------------------------------------------------

# CHOLMOD/GPU Module.  Copyright (C) 2005-2022, Timothy A. Davis.
# All Rights Reserved.
# SPDX-License-Identifier: GPL-2.0+

#-------------------------------------------------------------------------------
# get the version
#-------------------------------------------------------------------------------

# cmake 3.22 is required to find the BLAS/LAPACK
cmake_minimum_required ( VERSION 3.20 )

message ( STATUS "Building CHOLMOD_CUDA version: v"
    ${CHOLMOD_VERSION_MAJOR}.
    ${CHOLMOD_VERSION_MINOR}.
    ${CHOLMOD_VERSION_SUB} " (" ${CHOLMOD_DATE} ")" )

include ( SuiteSparsePolicy )

enable_language ( CUDA )
set ( CMAKE_CUDA_FLAGS "-cudart=static -lineinfo -DSUITESPARSE_CUDA" )
set ( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DSUITESPARSE_CUDA" )
message ( STATUS "C++ flags for CUDA:  ${CMAKE_CXX_FLAGS}" )
message ( STATUS "nvcc flags for CUDA: ${CMAKE_CUDA_FLAGS}" )
file ( GLOB CHOLMOD_CUDA_SOURCES "cholmod_gpu.c" "cholmod_l_gpu.c"
    "cholmod_gpu_kernels.cu" "../Utility/cholmod_error.c" "../Utility/cholmod_l_error.c" )

if ( BUILD_SHARED_LIBS )
    target_sources ( CHOLMOD PRIVATE ${CHOLMOD_CUDA_SOURCES} )
endif ( )

if ( BUILD_STATIC_LIBS )
    target_sources ( CHOLMOD_static PRIVATE ${CHOLMOD_CUDA_SOURCES} )
endif ( )

set ( CHOLMOD_CUDA_INCLUDES
        ../Check
        ../../SuiteSparse_config
        ../Cholesky
        ../Utility
        ../Supernodal
        ../Include
        ../GPU )

include_directories ( ${CHOLMOD_CUDA_INCLUDES} ${CUDAToolkit_INCLUDE_DIRS} )

if ( BUILD_SHARED_LIBS )
    target_include_directories ( CHOLMOD PRIVATE
        ${CUDAToolkit_INCLUDE_DIRS}
        ${CHOLMOD_CUDA_INCLUDES} )
    set_target_properties ( CHOLMOD PROPERTIES POSITION_INDEPENDENT_CODE ON )
    set_target_properties ( CHOLMOD PROPERTIES CUDA_SEPARABLE_COMPILATION ON )
    target_compile_definitions ( CHOLMOD PUBLIC "SUITESPARSE_CUDA" )
endif ( )

if ( BUILD_STATIC_LIBS )
    target_include_directories ( CHOLMOD_static PRIVATE
        ${CUDAToolkit_INCLUDE_DIRS}
        ${CHOLMOD_CUDA_INCLUDES} )
    set_target_properties ( CHOLMOD_static PROPERTIES CUDA_SEPARABLE_COMPILATION on )
    set_target_properties ( CHOLMOD_static PROPERTIES POSITION_INDEPENDENT_CODE on )
    target_compile_definitions ( CHOLMOD_static PUBLIC "SUITESPARSE_CUDA" )
endif ( )

if ( BUILD_SHARED_LIBS )
    target_link_libraries ( CHOLMOD PRIVATE
        CUDA::nvrtc CUDA::cudart_static CUDA::nvtx3 CUDA::cublas SuiteSparse::SuiteSparseConfig )
endif ( )
if ( BUILD_STATIC_LIBS )
    target_link_libraries ( CHOLMOD_CUDA_static PUBLIC CUDA::nvtx3 CUDA::cudart_static
        CUDA::nvToolsExt CUDA::cublas )
        if ( TARGET SuiteSparse::SuiteSparseConfig_static )
            target_link_libraries ( CHOLMOD_CUDA_static PUBLIC SuiteSparse::SuiteSparseConfig_static )
        else ( )
            target_link_libraries ( CHOLMOD_CUDA_static PUBLIC SuiteSparse::SuiteSparseConfig )
        endif ( )
endif ( )

#-------------------------------------------------------------------------------
# installation location
#-------------------------------------------------------------------------------

include ( CMakePackageConfigHelpers )

if ( BUILD_SHARED_LIBS )
    install ( TARGETS CHOLMOD_CUDA
        EXPORT CHOLMOD_CUDATargets
        LIBRARY DESTINATION ${SUITESPARSE_LIBDIR}
        ARCHIVE DESTINATION ${SUITESPARSE_LIBDIR}
        RUNTIME DESTINATION ${SUITESPARSE_BINDIR}
        PUBLIC_HEADER DESTINATION ${SUITESPARSE_INCLUDEDIR} )
endif ( )
if ( BUILD_STATIC_LIBS )
    install ( TARGETS CHOLMOD_CUDA_static
        EXPORT CHOLMOD_CUDATargets
        ARCHIVE DESTINATION ${SUITESPARSE_LIBDIR}
        PUBLIC_HEADER DESTINATION ${SUITESPARSE_INCLUDEDIR} )
endif ( )

# create (temporary) export target file during build
export ( EXPORT CHOLMOD_CUDATargets
    NAMESPACE SuiteSparse::
    FILE ${CMAKE_CURRENT_BINARY_DIR}/../CHOLMOD_CUDATargets.cmake )

# install export target, config and version files for find_package
install ( EXPORT CHOLMOD_CUDATargets
    NAMESPACE SuiteSparse::
    DESTINATION ${SUITESPARSE_PKGFILEDIR}/cmake/CHOLMOD_CUDA )

configure_package_config_file (
    ../Config/CHOLMOD_CUDAConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/../CHOLMOD_CUDAConfig.cmake
    INSTALL_DESTINATION ${SUITESPARSE_PKGFILEDIR}/cmake/CHOLMOD_CUDA )

write_basic_package_version_file (
    ${CMAKE_CURRENT_BINARY_DIR}/../CHOLMOD_CUDAConfigVersion.cmake
    COMPATIBILITY SameMajorVersion )

install ( FILES
    ${CMAKE_CURRENT_BINARY_DIR}/../CHOLMOD_CUDAConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/../CHOLMOD_CUDAConfigVersion.cmake
    DESTINATION ${SUITESPARSE_PKGFILEDIR}/cmake/CHOLMOD_CUDA )

#-------------------------------------------------------------------------------
# create pkg-config file
#-------------------------------------------------------------------------------

if ( NOT MSVC )
    set ( prefix "${CMAKE_INSTALL_PREFIX}" )
    set ( exec_prefix "\${prefix}" )
    cmake_path ( IS_ABSOLUTE SUITESPARSE_LIBDIR SUITESPARSE_LIBDIR_IS_ABSOLUTE )
    if (SUITESPARSE_LIBDIR_IS_ABSOLUTE)
        set ( libdir "${SUITESPARSE_LIBDIR}")
    else ( )
        set ( libdir "\${exec_prefix}/${SUITESPARSE_LIBDIR}")
    endif ( )
    cmake_path ( IS_ABSOLUTE SUITESPARSE_INCLUDEDIR SUITESPARSE_INCLUDEDIR_IS_ABSOLUTE )
    if (SUITESPARSE_INCLUDEDIR_IS_ABSOLUTE)
        set ( includedir "${SUITESPARSE_INCLUDEDIR}")
    else ( )
        set ( includedir "\${prefix}/${SUITESPARSE_INCLUDEDIR}")
    endif ( )
    configure_file (
        ../Config/CHOLMOD_CUDA.pc.in
        CHOLMOD_CUDA.pc
        @ONLY
        NEWLINE_STYLE LF )
    install ( FILES
        ${CMAKE_CURRENT_BINARY_DIR}/CHOLMOD_CUDA.pc
        DESTINATION ${SUITESPARSE_PKGFILEDIR}/pkgconfig )
endif ( )
