cmake_minimum_required(VERSION 3.15)

# The project name is defined to be the controller directory name.
get_filename_component(PROJECT ${CMAKE_SOURCE_DIR} NAME)
project(${PROJECT} LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

find_package(WebotsCtrl REQUIRED)

add_executable(${PROJECT})
target_link_libraries(${PROJECT} WebotsCtrl::CppController)

target_sources(${PROJECT} PRIVATE
  ${PROJECT}.cpp
)

# Copy the target executable at the right location.
add_custom_command(TARGET ${PROJECT} POST_BUILD COMMAND ${CMAKE_COMMAND} -E
  copy $<TARGET_FILE:${PROJECT}> ${CMAKE_SOURCE_DIR}
)

# Specify the default startup project for Visual Studio
set_property(DIRECTORY ${CMAKE_SOURCE_DIR} 
  PROPERTY VS_STARTUP_PROJECT ${PROJECT})
