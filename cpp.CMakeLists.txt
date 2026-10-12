cmake_minimum_required(VERSION 3.15)

project(CampusRouteFinder LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(CampusRouteFinder main.cpp)

if(WIN32)
    target_link_libraries(CampusRouteFinder PRIVATE ws2_32)
endif()