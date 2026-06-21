cmake_minimum_required(VERSION 3.16)
project(ShadowMapping LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

set(THIRD_PARTY_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../Libraries")
set(THIRD_PARTY_INCLUDE_DIR "${THIRD_PARTY_ROOT}/include")
set(THIRD_PARTY_LIB_DIR "${THIRD_PARTY_ROOT}/lib")
set(GLFW_LIBRARY "${THIRD_PARTY_LIB_DIR}/glfw3.lib")

find_package(OpenGL REQUIRED)

add_executable(${PROJECT_NAME} 
    src/main.cpp 
    vendor/glad/glad.c
)

target_include_directories(${PROJECT_NAME} PRIVATE 
    ${THIRD_PARTY_INCLUDE_DIR}
)

target_link_libraries(${PROJECT_NAME} PRIVATE 
    "${GLFW_LIBRARY}"
    OpenGL::GL
)

if(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE gdi32 user32 shell32)
endif()
