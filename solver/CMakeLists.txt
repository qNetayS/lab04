cmake_minimum_required(VERSION 3.10)
project(solver)

set(CMAKE_CXX_STANDARD 11)
add_executable(solver main.cpp)

target_include_directories(solver PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib
)

target_link_libraries(solver PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_ex/build/libformatter_ex.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib/build/libformatter.a
    ${CMAKE_CURRENT_SOURCE_DIR}/../solver_lib/build/libsolver_lib.a
)
