# ----------------------------------------------------------------------------
#  Root CMake file for ML Project
# ----------------------------------------------------------------------------
#
# Example CMake build line for windows & VS2015
# windows w/cuda: cmake -G "Visual Studio 14 2015 Win64" -DUSE_AVX_INSTRUCTIONS=ON ..
# linux w/o cuda: cmake -DUSE_AVX_INSTRUCTIONS=ON -DDLIB_USE_CUDA=OFF ..
#
#
# To compile:
# windows: cmake --build . --config Release
# linux: cmake --build . --config Release -- -j10
#


cmake_minimum_required(VERSION 2.8.12)

PROJECT(obj_det_run)

set(PLATFORM "$ENV{PLATFORM}" CACHE INTERNAL "Copied from environment variable")
if("${PLATFORM}" STREQUAL "")
    set(PLATFORM "NOT_SET")
    message("The PLATFORM Environment variable is not set on this system.  Setting it to:")
endif()

message("Platform: " ${PLATFORM})

option(USE_AVX_INSTRUCTIONS  "Compile the program with AVX instructions"  ON)
option(WITH_OBJ_LIB  "Compile the program using the object detection library"  ON)

if(WIN32)
    if(${PLATFORM} STREQUAL "Laptop")
        set(COMMON_ROOT "D:/Common")
        set(DLIB_CONTRIB "D:/Projects/dlib-contrib")
        include(D:/dlib-19.19/dlib/cmake)
    elseif(${PLATFORM} STREQUAL "LaptopN")
        set(COMMON_ROOT "C:/Projects/Common")
        set(DLIB_CONTRIB "C:/Projects/dlib-contrib")
        include(C:/dlib-19.19/dlib/cmake)    
    elseif(${PLATFORM} STREQUAL "BEAST")
        set(COMMON_ROOT "D:/Common")
        set(DLIB_CONTRIB "D:/Projects/dlib-contrib")
        include(D:/dlib-19.19/dlib/cmake)
    else()
        set(COMMON_ROOT "D:/Common")
        set(DLIB_CONTRIB "D:/Projects/dlib-contrib")
        include(D:/dlib-19.19/dlib/cmake)
    endif()    
else()
    if(${PLATFORM} STREQUAL "HPC")
        set(COMMON_ROOT "~/Projects/Common")
        set(DLIB_CONTRIB "~/Projects/dlib-contrib")
        include(~/dlib-19.19/dlib/cmake)
    elseif(${PLATFORM} STREQUAL "MainGear")
        set(COMMON_ROOT "~/Projects/Common")
        set(DLIB_CONTRIB "~/Projects/dlib-contrib")
        include(~/dlib-19.18/dlib/cmake)
    else()
        set(COMMON_ROOT "~/Projects/Common")
        set(DLIB_CONTRIB "~/Projects/dlib-contrib")
        include(~/dlib-19.19/dlib/cmake)
    endif()
endif()

if(WITH_OBJ_LIB)

    if(WIN32)
        set(OBJ_DET_DIRS "D:/Projects/obj_det_lib/include")
        set(OBJ_DET_LIBS "D:/Projects/obj_det_lib/build/Release/obj_det.lib")     
    else()
        set(OBJ_DET_DIRS "~/Projects/obj_det_lib/include")
        set(OBJ_DET_LIBS "~/Projects/obj_det_lib/build/libobj_det.so")
    endif()

    add_definitions(-DUSE_OBJ_LIB)
    add_definitions(-DBUILD_LIB)

endif()

set(exe_name "obj_det")

# grab all of the *.cpp files in the src directory
file(GLOB SOURCES "src/*.cpp" 
    "${COMMON_ROOT}/src/*.cpp"
    "${DLIB_CONTRIB}/src/*.cpp"
    )

set(project_includes "include" 
    "${COMMON_ROOT}/include" 
    "${DLIB_CONTRIB}/include" 
    "${OBJ_DET_DIRS}"
    )

# find OpenCV
if(${PLATFORM} STREQUAL "HPC")
    message("HPC detected.  OpenCV will not be used.")
else()
    find_package(OpenCV REQUIRED)
    if (OpenCV_FOUND)
        set(project_includes ${project_includes} ${OpenCV_INCLUDE_DIRS})
    else()
        message("OpenCV not found, so we won't build the project.")
    endif()
endif()

message(" ")
message("--------------------------------------------------------")
message("Source Files: ")
foreach(S ${SOURCES})
    message("  " ${S})
endforeach()
message("--------------------------------------------------------")

message(" ")

message("--------------------------------------------------------")
message("Project Includes: ")
foreach(S ${project_includes})
    message("  " ${S})
endforeach()
message("--------------------------------------------------------")
message(" ")

include_directories(${project_includes})
add_executable(${exe_name} ${SOURCES})
target_link_libraries(${exe_name} dlib::dlib ${OpenCV_LIBS} ${OBJ_DET_LIBS})

if(WIN32)
    message("MSVC Version: " ${MSVC_VERSION})
    # uncomment for MSVC and cmake greater than 3.6.0
    if(MSVC AND (${CMAKE_VERSION} VERSION_GREATER "3.6.0"))
        target_compile_options(${exe_name} PRIVATE "/MP")
        set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${exe_name})
    endif()
endif()

message("Build Complete!")
message(" ")
