cmake_minimum_required(VERSION 3.2)

project(PGFlow3D)

if(EXISTS "${CMAKE_SOURCE_DIR}/.git")
  execute_process(
    COMMAND git rev-parse --abbrev-ref HEAD
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_BRANCH
    OUTPUT_STRIP_TRAILING_WHITESPACE)
  execute_process(
    COMMAND git rev-parse HEAD
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_COMMIT_HASH
    OUTPUT_STRIP_TRAILING_WHITESPACE)
  execute_process(
    COMMAND git rev-parse --short HEAD
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GIT_COMMIT_HASH_SHORT
    OUTPUT_STRIP_TRAILING_WHITESPACE)
else()
  set(GIT_BRANCH "")
  set(GIT_COMMIT_HASH "")
  set(GIT_COMMIT_HASH_SHORT "")
endif()

message(STATUS "On Git Branch: ${GIT_BRANCH} (${GIT_COMMIT_HASH})")

set(DEFAULT_BUILD_TYPE "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(
    STATUS
      "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
  set(CMAKE_BUILD_TYPE
      "${DEFAULT_BUILD_TYPE}"
      CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui, ccmake
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif()
set(CMAKE_TRY_COMPILE_CONFIGURATION ${CMAKE_BUILD_TYPE})
set(CMAKE_CONFIGURATION_TYPES
    "Debug;Release"
    CACHE STRING "" FORCE)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS NO)


set(MSVC_COMPILER_FLAGS
    $<$<COMPILE_LANGUAGE:CXX>:/MP> # Use as many cores as available for
                                   # compilation.
    $<$<COMPILE_LANGUAGE:CXX>:/W4> # /Wall # Baseline reasonable warnings.
    $<$<COMPILE_LANGUAGE:CXX>:/permissive-> # Standards conformance mode for
                                            # MSVC compiler.
    $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:Release>:/Oi>>
    $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:Release>:/O2>>
    $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:Release>:/Ot>>
    $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:Release>:/GL>>
    $<$<COMPILE_LANGUAGE:CXX>:$<$<CONFIG:Release>:/GF>>)

set(CLANG_COMPILER_FLAGS
    -g
    -W
    -Wall
    -Wextra # Reasonable and standard.
    -Wshadow # Warn the user if a variable declaration shadows one from a parent
             # context.
    -Wnon-virtual-dtor # Warn the user if a class with virtual functions has a
                       # non-virtual destructor. This helps catch hard to track
                       # down memory errors.
    -Wcast-align # Warn for potential performance problem casts.
    -Wunused # Warn on anything being unused.
    -Wunused-function
    -Wunused-variable
    -Wunused-parameter
    -Woverloaded-virtual # Warn if you overload (not override) a virtual
                         # function.
    $<$<CONFIG:Debug>:-O0;-ggdb;-fno-elide-constructors>)

set(GCC_COMPILER_FLAGS
    ${CLANG_COMPILER_FLAGS}
)

if(MSVC)
  set(COMPILER_FLAGS ${MSVC_COMPILER_FLAGS})
elseif(CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")
  set(COMPILER_FLAGS ${CLANG_COMPILER_FLAGS})
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(COMPILER_FLAGS ${GCC_COMPILER_FLAGS})
else()
  message(
    AUTHOR_WARNING
      "No compiler flags set for '${CMAKE_CXX_COMPILER_ID}' compiler.")
endif()

file(WRITE "${PROJECT_BINARY_DIR}/.gitignore" "*")


find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

find_package(SFML COMPONENTS window graphics system REQUIRED)

add_executable(main Source/Main.cpp Source/Astar.cpp Source/BFS.cpp Source/Dijkstra.cpp Source/DrawMap.cpp Source/DrawText.cpp Source/GetAdjacentCells.cpp Source/GetCell.cpp Source/GetMouseCell.cpp)
target_link_libraries(main PRIVATE sfml-graphics)
target_compile_features(main PRIVATE cxx_std_17)
target_include_directories(main PUBLIC Source/Headers)
# target_link_libraries(PGFlow3D PRIVATE OpenMP::OpenMP_CXX)

if(MKL_FOUND)
    message(STATUS "MKL found")
    target_link_libraries(main PRIVATE MKL::MKL)
    target_compile_definitions(main PRIVATE EIGEN_USE_MKL_ALL)
endif()

target_compile_options(main PUBLIC ${COMPILER_FLAGS})



