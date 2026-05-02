cmake_minimum_required(VERSION 3.10)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
add_executable(hello_world main.cpp)

target_include_directories(hello_world PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
)

target_link_libraries(hello_world PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex/build/libformatter_ex.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib/build/libformatter.a
)
