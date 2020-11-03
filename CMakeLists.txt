cmake_minimum_required(VERSION 3.7) #Android variable is new in cmake version 3.7

set(n clinfo)
project(${n})

if(NOT ANDROID)
    if (DEFINED ENV{VCPKG_ROOT})
      set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
    endif ()

    find_package(OpenCL REQUIRED)

    add_executable(${n} src/${n}.c)
    target_include_directories(${n} PUBLIC src)
    target_include_directories(${n} PUBLIC ${OpenCL_INCLUDE_DIRS})
    target_link_libraries(${n} ${OpenCL_LIBRARIES} ${CMAKE_DL_LIBS})

    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT ${n})
else()
    #Get OpenCL headers directly
    include(FetchContent)
    FetchContent_Declare(
        OpenCLHeader
        GIT_REPOSITORY https://github.com/KhronosGroup/OpenCL-Headers.git
        GIT_TAG origin/master
    )
    FetchContent_MakeAvailable(OpenCLHeader)
    FetchContent_GetProperties(OpenCLHeader SOURCE_DIR OpenCLHeaderDir)
    include_directories(${OpenCLHeaderDir})

    #Link with OpenCL lib
    add_library(OpenCL SHARED IMPORTED)
    if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "aarch64")
        set(OpenCLLibLocation "/system/vendor/lib64/libOpenCL.so")
    else()
        set(OpenCLLibLocation "/system/vendor/lib/libOpenCL.so")
    endif()

    set_target_properties(OpenCL PROPERTIES IMPORTED_LOCATION ${OpenCLLibLocation})
    add_executable(${n} src/${n}.c)
    target_link_libraries(${n} PUBLIC OpenCL)
endif()
