cmake_minimum_required(VERSION 3.20)
project(gSOAP)

set(CMAKE_CXX_STANDARD 14)

add_executable(gSOAP main.cpp)

include_directories(${PROJECT_SOURCE_DIR})

link_libraries(crypto ssl z)
