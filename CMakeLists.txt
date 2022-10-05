cmake_minimum_required(VERSION 2.6)
project(CLAPACK C)
enable_testing()
include(CTest)

cmake_policy(SET CMP0026 OLD)
if(WIN32 AND NOT CYGWIN)
  set(SECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/winsecond.c)
  set(DSECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/windsecnd.c)
  add_definitions(-DNO_ISATTY -DMSDOS -DUSE_CLOCK)
else()
  set(SECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/second.c)
  set(DSECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/dsecnd.c)
endif()
enable_testing()
option(USE_BLAS_WRAP "pre-pend f2c_ to each function in blas" OFF)
if(NOT USE_BLAS_WRAP)
# _zrotg_ seems to be missing in the wrap header
  add_definitions(-DNO_BLAS_WRAP)
endif()

# lib install macro
macro( install_library TARGET_NAME )
  target_include_directories(${TARGET_NAME} PUBLIC
      $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )

  set_target_properties(${TARGET_NAME} PROPERTIES
      PREFIX ""
      LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
      RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
      ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
      PDB_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
      COMPILE_PDB_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
      PDB_NAME ${TARGET_NAME}
      COMPILE_PDB_NAME ${TARGET_NAME})

  install(TARGETS ${TARGET_NAME}
      EXPORT ${PROJECT_NAME}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

  if(MSVC)
      install(FILES ${CMAKE_BINARY_DIR}/Debug/${TARGET_NAME}.pdb DESTINATION $<IF:$<BOOL:${BUILD_SHARED_LIBS}>,${CMAKE_INSTALL_BINDIR},${CMAKE_INSTALL_LIBDIR}> CONFIGURATIONS Debug)
  endif(MSVC)
endmacro( install_library )

# set compiler options (run once)
if( NOT SETUP_COMPILER_FLAGS_HAS_RUN )
  if(MSVC) # silence some warnings
    set( WARN_FLAGS "${WARN_FLAGS} /wd4244" ) # conversion from 'doublereal' to 'real', possible loss of data
    set( WARN_FLAGS "${WARN_FLAGS} /wd4554" ) # check operator precedence for possible error; use parentheses to clarify precedence
    set( WARN_FLAGS "${WARN_FLAGS} /wd4996" ) # This function or variable may be unsafe. Consider using freopen_s instead.
    set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${WARN_FLAGS}"
         CACHE STRING "Flags used by the C compiler during all build types." FORCE )
  endif(MSVC)
  set( SETUP_COMPILER_FLAGS_HAS_RUN TRUE CACHE INTERNAL "one-time setup of compiler options" FORCE)
endif( NOT SETUP_COMPILER_FLAGS_HAS_RUN )

include_directories(${CLAPACK_SOURCE_DIR}/INCLUDE)
add_subdirectory(INCLUDE)
add_subdirectory(F2CLIBS)
add_subdirectory(BLAS)
add_subdirectory(SRC)
#add_subdirectory(TESTING) commented out because not working with android ndk r22 - redeclarations
set(CLAPACK_VERSION 3.2.1)
set(CPACK_PACKAGE_VERSION_MAJOR 3)
set(CPACK_PACKAGE_VERSION_MINOR 2)
set(CPACK_PACKAGE_VERSION_PATCH 1)
include(CPack)
install(EXPORT ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lapack FILE LAPACKConfig.cmake)
configure_file(${CLAPACK_SOURCE_DIR}/clapack-config-version.cmake.in
  ${CLAPACK_BINARY_DIR}/clapack-config-version.cmake @ONLY)
configure_file(${CLAPACK_SOURCE_DIR}/clapack-config.cmake.in
  ${CLAPACK_BINARY_DIR}/clapack-config.cmake @ONLY)
