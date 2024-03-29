# Create and configure a library target

# Find the dependencies

find_package(Boost REQUIRED)
find_package(fmt REQUIRED)
find_package(spdlog REQUIRED)

# Define the library type
if (UTILS_BUILD_SHARED_LIBS)
    set (UTILS_LIBRARY_TYPE SHARED)
    set(export_file_name "export_shared.h")
else ()
    set (UTILS_LIBRARY_TYPE STATIC)
    set(export_file_name "export_static.h")
endif ()

add_library (${PROJECT_NAME} ${UTILS_LIBRARY_TYPE})
add_library (wh::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# Describing the list of source files

NestingFileSearch (UTILS_HEADERS
    RECURSIVE
    EXTENSIONS ".h"
    DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/include"
    EXCLUDE "*.in"
)

NestingFileSearch (UTILS_SOURCES
    RECURSIVE
    EXTENSIONS ".cc"
    DIRECTORIES "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

target_sources (${PROJECT_NAME}
    PRIVATE
        ${UTILS_SOURCES}
        ${UTILS_HEADERS}
)

target_include_directories (${PROJECT_NAME}
    PUBLIC 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

target_link_libraries (${PROJECT_NAME}
    PUBLIC
        fmt::fmt
        Boost::boost
    PRIVATE
        spdlog::spdlog
)


# Create an installation target

if (UTILS_INSTALL AND NOT CMAKE_SKIP_INSTALL_RULES)
    include (GNUInstallDirs)
    include (CMakePackageConfigHelpers)

    set (UTILS_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
    set (UTILS_PKGCONFIG_DIR ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
    set (VERSION_CONFIG ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake)
    set (PROJECT_CONFIG ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake)
    set (UTILS_TARGETS_EXPORT_NAME ${PROJECT_NAME}-targets)

    # Generate the version, config and target files into the build directory.
    write_basic_package_version_file (
        ${VERSION_CONFIG}
        VERSION ${UTILS_VERSION}
        COMPATIBILITY AnyNewerVersion
    )
    
    configure_file (
        "${PROJECT_SOURCE_DIR}/cmake/utils.pc.in"
        "${UTILS_PKGCONFIG_DIR}"
        @ONLY
    )

    configure_package_config_file (
        ${PROJECT_SOURCE_DIR}/cmake/utils-config.cmake.in
        ${PROJECT_CONFIG}
        INSTALL_DESTINATION ${UTILS_CMAKE_DIR}
    )

    # Install the library
    install (
        TARGETS ${PROJECT_NAME} 
        EXPORT ${UTILS_TARGETS_EXPORT_NAME}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        FRAMEWORK DESTINATION "."
        INCLUDES
    )

    # Install headers.
    install (
        DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    # Use a namespace because CMake provides better diagnostics for namespaced imported targets.
    export (
        TARGETS ${PROJECT_NAME}
        NAMESPACE wh::
        FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-targets.cmake
    )

    # Install version, config and target files.
    install (
        FILES ${PROJECT_CONFIG} ${VERSION_CONFIG}
        DESTINATION ${UTILS_CMAKE_DIR}
    )

    install (
        EXPORT ${PROJECT_NAME}-targets 
        DESTINATION ${UTILS_CMAKE_DIR}
        NAMESPACE wh::
    )
    
    # $<TARGET_PDB_FILE:tgt> generates an error if the library is built with the /MT flag
    # so the path to the pdb file is created from the available variables
    install (
        FILES $<TARGET_FILE_DIR:${PROJECT_NAME}>/$<TARGET_FILE_BASE_NAME:${PROJECT_NAME}>.pdb
        DESTINATION ${CMAKE_INSTALL_LIBDIR}
        OPTIONAL
    )
endif ()
