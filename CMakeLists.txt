# Root CMakeLists.txt for template C++ project
# Author: Jesper Glas

# Set minimum cmake version
cmake_minimum_required(VERSION 3.16)

include(FetchContent)

# Set project parameters
project(
    "cpp-simsolar"
    VERSION 1.0
)

# Compiler config
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED on)
set(CMAKE_CXX_FLAGS "-Wall -Werror")

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

# Set variables for src and include directories
set(SOURCE ${PROJECT_SOURCE_DIR}/src)
set(INCLUDE ${PROJECT_SOURCE_DIR}/include)

# Include project config header
configure_file(
    ${INCLUDE}/ProjectConfig.hpp.in
    ProjectConfig.hpp
)

# Variable for linked libraries
set(PROJECT_LIBRARIES)

# Define variable for linked libraries
set(PROJECT_LIBRARIES)

# OpenGL
message(STATUS "Checking OpenGL status..")
find_package(OpenGL REQUIRED)
if(OPENGL_FOUND)
  include_directories(SYSTEM ${OPENGL_INCLUDE_DIR})
  set(PROJECT_LIBRARIES ${PROJECT_LIBRARIES} ${OPENGL_LIBRARIES})
else()
  message(FATAL_ERROR "OpenGL not found!")
endif(OPENGL_FOUND)

# GLFW (used for window handling)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/glfw ${CMAKE_CURRENT_BINARY_DIR}/glfw)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/glfw/include)
set(PROJECT_LIBRARIES ${PROJECT_LIBRARIES} ${GLFW_LIBRARIES})

# gl3w (used for OpenGL function loading)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/external/gl3w/src PROJECT_SRCS)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/gl3w/include)

# GLM (used for math functions and matrix/vector types)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/glm)

# Dear ImGui (used for GUI)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/external/imgui PROJECT_SRCS)
set(PROJECT_SRCS ${PROJECT_SRCS}
  ${CMAKE_CURRENT_SOURCE_DIR}/external/imgui/examples/imgui_impl_glfw.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/external/imgui/examples/imgui_impl_opengl3.cpp)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/imgui)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/imgui/examples)
add_definitions(-DIMGUI_IMPL_OPENGL_LOADER_GL3W)

# stb_image (used for image I/O)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/stb)

# RapidJSON (used for JSON I/O)
include_directories(SYSTEM ${CMAKE_CURRENT_SOURCE_DIR}/external/rapidjson/include)

# Set extra compiler flags
if(UNIX AND NOT APPLE)
  set(CMAKE_CXX_FLAGS "-W -Wall -std=c++11")
endif(UNIX AND NOT APPLE)
if(APPLE)
  set(CMAKE_CXX_FLAGS "-W -Wall -std=c++11 -ObjC++")
endif(APPLE)

# Enable testing
enable_testing()

# Add subdirs
add_subdirectory(lib)
add_subdirectory(tests)

# Make app executable
add_executable(
    ${PROJECT_NAME}
    ${PROJECT_SRCS}
    ${SOURCE}/main.cpp
    ${SOURCE}/base.cpp
    ${SOURCE}/cg_trackball.cpp
    ${SOURCE}/attribute.cpp
)

# Set main target include directories
target_include_directories(
    ${PROJECT_NAME}
    PUBLIC
        ${PROJECT_BINARY_DIR}
        ${INCLUDE}
)

# Link libraries
target_link_libraries(
    ${PROJECT_NAME}
        glfw
        ${PROJECT_LIBRARIES}
)