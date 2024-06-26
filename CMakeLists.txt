cmake_minimum_required (VERSION 3.10)
project(incppect)

#[[ 
  Suppress Warning for compatibility with older versions of CMake.
  The option is clearing the variable 'INCPPECT_DEBUG'.
#]]
cmake_policy(SET CMP0077 NEW)

macro(hide_warnings)
    if (MSVC)
        # Suppress warning C4996 (deprecated declarations) for Visual Studio
        add_compile_options(/wd4996 /wd4100 /wd4267 /wd4996)
    elseif (CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_CLANG)
        # Suppress warnings about deprecated OpenSSL functions and unused parameters for GCC and Clang
        add_compile_options(
            -Wno-deprecated-declarations
            -Wno-sign-compare
            -Wno-unused-parameter
            -Wno-unused-variable 
    )
    endif()
endmacro()

set(template_path "${CMAKE_CURRENT_SOURCE_DIR}/include/localhost-root-path.in.hpp")
get_filename_component(template_path "${template_path}" ABSOLUTE)

macro(assign_local_host_root_path project_folder)
    set(http_root_dir "\"${CMAKE_CURRENT_SOURCE_DIR}\"")
    set(output_path "${CMAKE_CURRENT_SOURCE_DIR}/localhost-root-path.hpp")
    configure_file("${template_path}" "${output_path}" @ONLY)
endmacro()


hide_warnings()

if (IMGUI_WS_ALL_WARNINGS AND (CMAKE_COMPILER_IS_GNUCC OR CMAKE_CXX_COMPILER_ID MATCHES "Clang"))
	# Enable all warnings for GCC and Clang
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")
elseif (IMGUI_WS_ALL_WARNINGS)
	message(WARNING "imgui-ws is not supported on Windows!")
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS "on")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

if(NOT PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
   include("./cmake/uwebsockets.cmake")
endif()

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo")
endif ()

option(INCPPECT_DEBUG   "Enable debug messages in the incppect service" OFF)
option(INCPPECT_NO_SSL  "Disable SSL support" ON)

include(FetchContent)

FetchContent_Declare(
  glaze
  GIT_REPOSITORY https://github.com/stephenberry/glaze.git
  GIT_TAG main
  GIT_SHALLOW TRUE
)

FetchContent_MakeAvailable(glaze)

find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)

if (APPLE)
    include(./cmake/FindLibUV.cmake)
    find_package(libuv REQUIRED)
    message(STATUS "libuv include directories: ${LIBUV_INCLUDE_DIRS}")
    message(STATUS "libuv libraries: ${LIBUV_LIBRARIES}")
    message(STATUS "libuv version: ${LIBUV_VERSION}")
endif()

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/js/incppect.js" src_incppect_js)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/incppect/common.h.in ${CMAKE_CURRENT_SOURCE_DIR}/include/incppect/common.h @ONLY)

add_library(incppect_incppect INTERFACE)

add_library(incppect::incppect ALIAS incppect_incppect)

set_property(TARGET incppect_incppect PROPERTY EXPORT_NAME incppect)

target_include_directories(
    incppect_incppect
    INTERFACE 
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/../imgui/imgui>"
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/../imgui/imgui-extra>"
)

target_include_directories(incppect_incppect INTERFACE)

find_uwebsockets()

target_link_libraries(incppect_incppect INTERFACE uWS glaze::glaze)

if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    add_subdirectory(examples)
endif ()
