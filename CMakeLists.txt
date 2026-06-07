cmake_minimum_required(VERSION 3.11)
project(raylib_game C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# Build GLFW
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
add_subdirectory(raylib/src/external/glfw glfw)

# Build raylib library from source files
file(GLOB RAYLIB_SOURCES 
    "raylib/src/rcore.c" 
    "raylib/src/rshapes.c" 
    "raylib/src/rtext.c" 
    "raylib/src/rtextures.c" 
    "raylib/src/raudio.c" 
    "raylib/src/rmodels.c"
)

# Create raylib library
add_library(raylib_lib STATIC ${RAYLIB_SOURCES})

# Set platform-specific defines BEFORE including headers
if(APPLE)
    target_compile_definitions(raylib_lib PRIVATE 
        PLATFORM_DESKTOP 
        GRAPHICS_API_OPENGL_33
    )
    target_link_libraries(raylib_lib PRIVATE "-framework IOKit" "-framework Cocoa" "-framework OpenGL")
elseif(UNIX AND NOT APPLE)
    target_compile_definitions(raylib_lib PRIVATE 
        PLATFORM_DESKTOP 
        GRAPHICS_API_OPENGL_33
        _POSIX_C_SOURCE=200112L
    )
    target_link_libraries(raylib_lib PRIVATE X11 m)
elseif(WIN32)
    target_compile_definitions(raylib_lib PRIVATE 
        PLATFORM_DESKTOP 
        GRAPHICS_API_OPENGL_33
    )
    target_link_libraries(raylib_lib PRIVATE gdi32 winmm)
endif()

target_include_directories(raylib_lib 
    PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}/raylib/src
    ${CMAKE_CURRENT_SOURCE_DIR}/raylib/src/external
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/raylib/src/external/glfw/include
)

target_link_libraries(raylib_lib PRIVATE glfw)

# Create main executable
add_executable(game src/main.cpp)
target_link_libraries(game PRIVATE raylib_lib)

# Link math library on non-Windows
if(NOT WIN32)
    target_link_libraries(game PRIVATE m)
endif()
