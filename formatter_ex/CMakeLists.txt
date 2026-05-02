add_library(formatter_ex STATIC formatter_ex.cpp)
target_link_libraries(formatter_ex formatter)
target_include_directories(formatter_ex PUBLIC .)
