cmake_minimum_required(VERSION 3.19)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(macdeployqt)

find_package(Qt6 REQUIRED COMPONENTS Core)

add_executable(macdeployqt
    src/main.cpp
    src/shared.cpp
    src/shared.h
    )

target_link_libraries(macdeployqt
    PRIVATE
    Qt6::Core
    "-framework CoreFoundation"
    )

