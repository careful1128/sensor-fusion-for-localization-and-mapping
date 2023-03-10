# config file support for find_package (GeographicLib).  This needs to
# deal with two environments: (1) finding the build tree and (2)
# finding the install tree.  geographiclib-config.cmake detects which
# situation it is handing by looking at @PROJECT_ROOT_DIR@.  If
# this is an absolute path, it's in the build tree; otherwise, it's in the
# install tree.  (Note that the whole install tree can be relocated.)

# Variables needed by ${PROJECT_NAME_LOWER}-config-version.cmake
if (MSVC)
  # For checking the compatibility of MSVC_TOOLSET_VERSION; see
  # https://docs.microsoft.com/en-us/cpp/porting/overview-of-potential-upgrade-issues-visual-cpp
  # Assume major version number is obtained by dropping the last decimal
  # digit.
  math (EXPR MSVC_TOOLSET_MAJOR "${MSVC_TOOLSET_VERSION}/10")
else ()
  set (MSVC_TOOLSET_VERSION 0)
  set (MSVC_TOOLSET_MAJOR 0)
endif ()
if (CMAKE_CROSSCOMPILING)
  # Ensure that all "true" (resp. "false") settings are represented by
  # the same string.
  set (CMAKE_CROSSCOMPILING_STR "ON")
else ()
  set (CMAKE_CROSSCOMPILING_STR "OFF")
endif ()

# geographiclib-config.cmake for the build tree
set (PROJECT_ROOT_DIR "${PROJECT_BINARY_DIR}")
set (PROJECT_INCLUDE_DIRS
  "${PROJECT_BINARY_DIR}/include" "${PROJECT_SOURCE_DIR}/include")

if (PROJECT_STATIC_LIBRARIES)
  set (CONFIG_STATIC_LIBRARIES "${PROJECT_NAME}::${PROJECT_STATIC_LIBRARIES}")
else ()
  set (CONFIG_STATIC_LIBRARIES)
endif ()
if (PROJECT_SHARED_LIBRARIES)
  set (CONFIG_SHARED_LIBRARIES "${PROJECT_NAME}::${PROJECT_SHARED_LIBRARIES}")
else ()
  set (CONFIG_SHARED_LIBRARIES)
endif ()

configure_file (project-config.cmake.in
  "${PROJECT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config.cmake" @ONLY)
configure_file (project-config-version.cmake.in
  "${PROJECT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config-version.cmake" @ONLY)
export (TARGETS
  ${PROJECT_ALL_LIBRARIES} ${TOOLS}
  FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME_LOWER}-legacy-targets.cmake")
export (TARGETS
  ${PROJECT_ALL_LIBRARIES} ${TOOLS}
  NAMESPACE ${PROJECT_NAME}::
  FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME_LOWER}-targets.cmake")

# geographiclib-config.cmake for the install tree.  It's installed in
# ${INSTALL_CMAKE_DIR} and @PROJECT_ROOT_DIR@ is the relative
# path to the root from there.  (Note that the whole install tree can
# be relocated.)
if (COMMON_INSTALL_PATH)
  # Install under lib${LIB_SUFFIX} so that 32-bit and 64-bit packages
  # can be installed on a single machine.
  set (INSTALL_CMAKE_DIR "lib${LIB_SUFFIX}/cmake/${PROJECT_NAME}")
else ()
  set (INSTALL_CMAKE_DIR "cmake")
endif ()
# Find root of install tree relative to INSTALL_CMAKE_DIR
file (RELATIVE_PATH PROJECT_ROOT_DIR
  "${CMAKE_INSTALL_PREFIX}/${INSTALL_CMAKE_DIR}" "${CMAKE_INSTALL_PREFIX}")
# strip trailing slash
get_filename_component (PROJECT_ROOT_DIR "${PROJECT_ROOT_DIR}/." PATH)

# @PROJECT_INCLUDE_DIRS@ is not used in the install tree; reset
# it to prevent the source and build paths appearing in the installed
# config files
set (PROJECT_INCLUDE_DIRS)
configure_file (project-config.cmake.in project-config.cmake @ONLY)
configure_file (project-config-version.cmake.in
  project-config-version.cmake @ONLY)
install (FILES
  "${CMAKE_CURRENT_BINARY_DIR}/project-config.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}"
  RENAME "${PROJECT_NAME_LOWER}-config.cmake")
install (FILES
  "${CMAKE_CURRENT_BINARY_DIR}/project-config-version.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}"
  RENAME "${PROJECT_NAME_LOWER}-config-version.cmake")
# Make information about the cmake targets (the library and the tools)
# available.
install (EXPORT targets
  FILE ${PROJECT_NAME_LOWER}-legacy-targets.cmake
  DESTINATION "${INSTALL_CMAKE_DIR}")
install (EXPORT targets
  NAMESPACE ${PROJECT_NAME}::
  FILE ${PROJECT_NAME_LOWER}-targets.cmake
  DESTINATION "${INSTALL_CMAKE_DIR}")

if (MSVC AND PACKAGE_DEBUG_LIBS)
  install (FILES
    "${PROJECT_BINARY_DIR}/cmake/CMakeFiles/Export/cmake/${PROJECT_NAME_LOWER}-targets-debug.cmake"
    DESTINATION "${INSTALL_CMAKE_DIR}" CONFIGURATIONS Release)
endif ()

# Support for pkgconfig/geographiclib.pc
set (prefix ${CMAKE_INSTALL_PREFIX})
set (exec_prefix "\${prefix}")
set (libdir "\${exec_prefix}/lib${LIB_SUFFIX}")
set (includedir "\${prefix}/include")
set (bindir "\${exec_prefix}/bin")
set (PACKAGE_NAME "${PROJECT_NAME}")
set (PACKAGE_VERSION "${PROJECT_VERSION}")

configure_file (project.pc.in geographiclib.pc @ONLY)
install (FILES
  "${CMAKE_CURRENT_BINARY_DIR}/geographiclib.pc"
  DESTINATION "lib${LIB_SUFFIX}/pkgconfig")
