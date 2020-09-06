cmake_minimum_required(VERSION 3.16)

list(PREPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/)

include(CMakePrintHelpers)
include(cmtk/CppLibraryProject)

#-----
# PROJECT

check_cmake_binary_dir()
set_build_type_if_undefined()

#-----
# C++ PROJECT

project(seri
        VERSION 0.1.0
#        DESCRIPTION ""
#        HOMEPAGE_URL ""
        LANGUAGES CXX)

message(STATUS "BUILD   : ${CMAKE_BUILD_TYPE}")
message(STATUS "CPPCOMP : ${CMAKE_CXX_COMPILER} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "PROJECT : ${PROJECT_NAME} ${PROJECT_VERSION}")

#-----
include(CTest)

# Project options
library_build_options(${PROJECT_NAME} STATIC SHARED EXAMPLE TEST)

# Headers:
set(headers
include/seri/polymorphism.hpp
include/seri/binary/binary_io.hpp
include/seri/binary/read_binary.hpp
include/seri/binary/write_binary.hpp
include/seri/archive/binary_archive.hpp
include/seri/archive/archive_base.hpp
)

# Sources:
set(sources
src/seri.cpp
)

# Add C++ library
add_cpp_library(${PROJECT_NAME} ${PROJECT_NAME}_BUILD_SHARED_LIB ${PROJECT_NAME}_BUILD_STATIC_LIB
    SHARED ${PROJECT_NAME}
    STATIC ${PROJECT_NAME}-static
    CXX_STANDARD 20
    INCLUDE_DIRECTORIES include
    OUTPUT_VERSION_HEADER "version.hpp"
    HEADERS ${headers}
    SOURCES ${sources}
    BUILT_TARGETS project_targets
    )

set(core_target core)
if(NOT TARGET core)
    string(PREPEND core_target core::)
    find_package(core 0.3.0 REQUIRED CONFIG)
endif()
foreach(target ${project_targets})
    target_link_libraries(${target} PUBLIC ${core_target})
endforeach()

# Install C++ library
install_cpp_library_targets(${PROJECT_NAME}
                            TARGETS ${project_targets}
                            INCLUDE_DIRECTORIES "include/${PROJECT_NAME}"
                            NAMESPACE ${PROJECT_NAME}::
                            )

# Install package
install_package(${PROJECT_NAME}
                VERBOSE_PACKAGE_CONFIG_FILE
                EXPORT_NAMES ${PROJECT_NAME})

if(${PROJECT_NAME}_BUILD_EXAMPLES)
    add_subdirectory(example)
endif()

if(${PROJECT_NAME}_BUILD_TESTS AND BUILD_TESTING)
    add_subdirectory(test)
endif()

#-----