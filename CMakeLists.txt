cmake_minimum_required(VERSION 3.16)
project(ShadowMappingSimple LANGUAGES C CXX)

# C++ standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Output folder
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Third-party paths
set(THIRD_PARTY_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/include")
set(THIRD_PARTY_LIB_DIR     "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/lib")

# Source files
set(SRC_MAIN "${CMAKE_CURRENT_SOURCE_DIR}/Main.cpp")
set(SRC_GLAD "${CMAKE_CURRENT_SOURCE_DIR}/glad.c")

# ---- GLAD ----
add_library(glad STATIC ${SRC_GLAD})

target_include_directories(glad PUBLIC
    ${THIRD_PARTY_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# ---- Executable ----
add_executable(${PROJECT_NAME} ${SRC_MAIN})

target_include_directories(${PROJECT_NAME} PRIVATE
    ${THIRD_PARTY_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(${PROJECT_NAME} PRIVATE glad)

# ---- OpenGL ----
find_package(OpenGL REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)

# ---- GLFW ----
find_library(GLFW_LIB
    NAMES glfw3 glfw
    PATHS ${THIRD_PARTY_LIB_DIR}
)

if(NOT GLFW_LIB)
    message(FATAL_ERROR "GLFW not found in ${THIRD_PARTY_LIB_DIR}")
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE ${GLFW_LIB})

# ---- Windows system libs ----
if(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        gdi32
        user32
        shell32
    )
endif()

# ---- Copy shaders ----
set(SHADERS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../shaders")

if(EXISTS ${SHADERS_DIR})
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${SHADERS_DIR}
        $<TARGET_FILE_DIR:${PROJECT_NAME}>/shaders
    )
endif()

# ---- MSVC warnings ----
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
endif()