cmake_minimum_required(VERSION 3.16)
project(ShadowMappingSimple LANGUAGES C CXX)

# Basic
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Output folders
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG   ${CMAKE_BINARY_DIR}/bin/Debug)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin/Release)

# Third-party paths (relative to this folder)
set(THIRD_PARTY_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/include" CACHE PATH "Third-party include dir")
set(THIRD_PARTY_LIB_DIR     "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries/lib"     CACHE PATH "Third-party lib dir")

# Source files
set(SRC_MAIN "${CMAKE_CURRENT_SOURCE_DIR}/Main.cpp")
set(SRC_GLAD "${CMAKE_CURRENT_SOURCE_DIR}/glad.c")

if (NOT EXISTS "${SRC_MAIN}")
    message(FATAL_ERROR "Main.cpp not found at ${SRC_MAIN}")
endif()
if (NOT EXISTS "${SRC_GLAD}")
    message(FATAL_ERROR "glad.c not found at ${SRC_GLAD}")
endif()

# Build glad as a static library (so its symbols are linked in)
add_library(glad STATIC "${SRC_GLAD}")
# Ensure glad sees includes
target_include_directories(glad PUBLIC ${THIRD_PARTY_INCLUDE_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

# Create executable
add_executable(${PROJECT_NAME} "${SRC_MAIN}")
target_include_directories(${PROJECT_NAME} PRIVATE ${THIRD_PARTY_INCLUDE_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

# Link glad into executable
target_link_libraries(${PROJECT_NAME} PRIVATE glad)

# OpenGL (system)
if (WIN32)
    find_package(OpenGL REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)
endif()

# GLFW: try to use local libs: glfw3d.lib (Debug) and glfw3.lib (Release) if present
if (EXISTS "${THIRD_PARTY_LIB_DIR}/glfw3.lib" OR EXISTS "${THIRD_PARTY_LIB_DIR}/glfw3d.lib")
    add_library(glfw3_imported UNKNOWN IMPORTED)
    if (EXISTS "${THIRD_PARTY_LIB_DIR}/glfw3d.lib")
        set_target_properties(glfw3_imported PROPERTIES IMPORTED_LOCATION_DEBUG "${THIRD_PARTY_LIB_DIR}/glfw3d.lib")
    endif()
    if (EXISTS "${THIRD_PARTY_LIB_DIR}/glfw3.lib")
        set_target_properties(glfw3_imported PROPERTIES IMPORTED_LOCATION_RELEASE "${THIRD_PARTY_LIB_DIR}/glfw3.lib"
                                                      IMPORTED_LOCATION "${THIRD_PARTY_LIB_DIR}/glfw3.lib")
    endif()
    target_link_libraries(${PROJECT_NAME} PRIVATE glfw3_imported)
else()
    message(FATAL_ERROR "No GLFW .lib found in ${THIRD_PARTY_LIB_DIR}. Expected glfw3.lib or glfw3d.lib")
endif()

# Copy dll only if exists
if (EXISTS "${THIRD_PARTY_LIB_DIR}/glfw3.dll")
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${THIRD_PARTY_LIB_DIR}/glfw3.dll"
            $<TARGET_FILE_DIR:${PROJECT_NAME}>
        COMMENT "Copying glfw3.dll to executable folder"
    )
endif()

# copy shaders if present
set(SHADERS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../shaders")
if (EXISTS "${SHADERS_DIR}")
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${SHADERS_DIR}" $<TARGET_FILE_DIR:${PROJECT_NAME}>/shaders
    )
endif()

# MSVC niceties
if (MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /WX-)
endif()
