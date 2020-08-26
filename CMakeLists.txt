CMAKE_MINIMUM_REQUIRED(VERSION 3.5)
PROJECT(lambdatwist-pnp CXX) # defines ${CMAKE_PROJECT_NAME}(top level) PROJECT_NAME(most recent)


macro(print_list name list)
    message("\n Listing: ${name}\n")
    foreach(item IN LISTS ${list})
        message("     ${name}: ${item}")
    endforeach()
    message("\n Listing: ${name} - done \n")
endmacro()

if(NOT CMAKE_BUILD_TYPE)
    MESSAGE("-- No build type specified; defaulting to CMAKE_BUILD_TYPE=Release.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING
            "Choose the type of build, options are: Debug Release"  FORCE)
else()
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        MESSAGE("\n${line}")
        MESSAGE("\n-- Build type: Debug. Performance will be terrible!")
        MESSAGE("-- Add -DCMAKE_BUILD_TYPE=Release to the CMake command line to get an optimized build.")
        MESSAGE("\n${line}")
    endif()
endif()
# add warnings
#set(WARN "-Wall -Wextra -Wpedantic -Werror")
set(CMAKE_CXX_FLAGS_DEBUG " -fno-omit-frame-pointer -g -pg  -rdynamic ${WARN} ") # dynamic is for the improved asserts
set(CMAKE_CXX_FLAGS_RELEASE " -O3 -march=native -DNDEBUG ${WARN} ")



option(RANDOM_SEED_FROM_TIME "generate a seed from time, default" OFF)
if(RANDOM_SEED_FROM_TIME)
    add_definitions(-DRANDOM_SEED_FROM_TIME)
else()
    add_definitions(-DRANDOM_SEED_VALUE=${RANDOM_SEED})
endif()

add_definitions("-std=c++11")
include_directories(".")
include_directories("lambdatwist-p3p/utils")

find_package(Ceres REQUIRED )
INCLUDE_DIRECTORIES(${CERES_INCLUDE_DIRS} .)

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH}")

#including headers here make them show up in older ides
file(GLOB_RECURSE lambdatwist-headers "lambdatwist/*")

#file(GLOB_RECURSE utils2 )


#list(APPEND util-srcs data.h)
#list(APPEND util-srcs solver.h)
#list(APPEND util-srcs mnumerics.h)
#list(APPEND util-srcs p3p_generator.h)

# headers are included for ide benefit...
add_library(lambdatwist SHARED src/mlibtime.cpp src/pnp_ransac.cpp src/p4p.cpp ${lambdatwist-headers})
target_link_libraries(lambdatwist ${CERES_LIBRARIES})


add_executable(main test_pnp.cpp simulator.h)
target_link_libraries(main lambdatwist)

message("================================================================================")


# Install step
install(TARGETS lambdatwist EXPORT lambdatwistConfig
    ARCHIVE  DESTINATION lib
    LIBRARY  DESTINATION lib
    RUNTIME  DESTINATION bin)
install(DIRECTORY lambdatwist DESTINATION include)

set(ConfigPackageLocation lib/cmake/lambdatwist)

set(CONF_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/lambdatwistConfig.cmake.in
"${PROJECT_BINARY_DIR}/lambdatwistConfig.cmake")

install (FILES
    ${CMAKE_CURRENT_BINARY_DIR}/lambdatwistConfig.cmake
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/cmake/lambdatwist")
