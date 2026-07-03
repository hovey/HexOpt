cmake_minimum_required(VERSION 3.20)
project(HexOpt CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Apple clang has no built-in OpenMP; install via `brew install libomp`.
if(APPLE AND NOT DEFINED OpenMP_ROOT)
  execute_process(
    COMMAND brew --prefix libomp
    OUTPUT_VARIABLE HOMEBREW_LIBOMP_PREFIX
    OUTPUT_STRIP_TRAILING_WHITESPACE
    ERROR_QUIET
  )
  if(HOMEBREW_LIBOMP_PREFIX)
    set(OpenMP_ROOT ${HOMEBREW_LIBOMP_PREFIX})
  endif()
endif()

find_package(OpenMP REQUIRED)

add_executable(HexOpt
  geometry.cpp
  meshQuality.cpp
  optimizer.cpp
)

target_link_libraries(HexOpt PRIVATE OpenMP::OpenMP_CXX)
