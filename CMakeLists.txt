cmake_minimum_required(VERSION 3.15)

set(version 1.0.0)

project(webots_ctrl VERSION ${version}
    DESCRIPTION "Webots controller" LANGUAGES CXX)

file(TO_CMAKE_PATH "$ENV{WEBOTS_HOME}" webots_home)

if (MSVC)

add_library(CController INTERFACE)

target_include_directories(CController INTERFACE
    ${webots_home}/include/controller/c
)

target_link_libraries(CController INTERFACE
    ${webots_home}/lib/controller/Controller.lib
)

target_compile_definitions(CController INTERFACE
    "_USE_MATH_DEFINES"
)

set(CMAKE_DEBUG_POSTFIX d)

add_library(CppController STATIC)

file(GLOB cpp_src ${webots_home}/src/controller/cpp/*.cpp)
target_sources(CppController PRIVATE ${cpp_src})

target_link_libraries(CppController PUBLIC CController)

target_include_directories(CppController PUBLIC
    ${webots_home}/include/controller/cpp
)


endif(MSVC)

if (UNIX)

add_library(CController INTERFACE)

target_include_directories(CController INTERFACE
    ${webots_home}/include/controller/c
)

target_link_libraries(CController INTERFACE
    ${webots_home}/lib/controller/libController.so
)

add_library(CppController INTERFACE)

target_include_directories(CppController INTERFACE
    ${webots_home}/include/controller/cpp
)

target_link_libraries(CppController INTERFACE
    ${webots_home}/lib/controller/libCppController.so
    CController
)

endif(UNIX)


install(TARGETS CController CppController 
    EXPORT WebotsCtrlConfig
    ARCHIVE DESTINATION lib
)

install(EXPORT WebotsCtrlConfig
    NAMESPACE WebotsCtrl::
    DESTINATION lib/cmake/WebotsCtrl
) 

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_BINARY_DIR}/WebotsCtrlConfigVersion.cmake
  VERSION ${version}
  COMPATIBILITY SameMajorVersion )

install(FILES 
    ${CMAKE_BINARY_DIR}/WebotsCtrlConfigVersion.cmake
    DESTINATION lib/cmake/WebotsCtrl
)



