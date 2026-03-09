cmake_minimum_required(VERSION 3.16)
project(ShadowMapping LANGUAGES C CXX)

# C++ version
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Output
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Third party paths
set(THIRD_PARTY_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/include")
set(THIRD_PARTY_LIB_DIR     "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/lib")

# Sources
set(SOURCES
    Source/Main.cpp
    Source/Shader.cpp
)

set(HEADERS
    Headers/Shader.h
)

# GLAD
add_library(glad STATIC glad.c)
target_include_directories(glad PUBLIC ${THIRD_PARTY_INCLUDE_DIR})

# Executable
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

target_include_directories(${PROJECT_NAME} PRIVATE
    Headers
    ${THIRD_PARTY_INCLUDE_DIR}
)

target_link_libraries(${PROJECT_NAME} PRIVATE glad)

# OpenGL
find_package(OpenGL REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)

# GLFW
find_library(GLFW_LIB
    NAMES glfw3 glfw
    PATHS ${THIRD_PARTY_LIB_DIR}
)

if(NOT GLFW_LIB)
    message(FATAL_ERROR "GLFW not found")
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE ${GLFW_LIB})

# Windows libs
if(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        gdi32
        user32
        shell32
    )
endif()

# Copy shaders
set(SHADERS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Shaders")

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${SHADERS_DIR}
    $<TARGET_FILE_DIR:${PROJECT_NAME}>/Shaders
)

# MSVC warnings
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
endif()