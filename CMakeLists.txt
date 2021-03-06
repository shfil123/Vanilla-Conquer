cmake_minimum_required(VERSION 3.10)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
include(FeatureSummary)

if(NOT DEFINED CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo" FORCE)
endif()

project(VanillaConquer CXX)

option(BUILD_REMASTERTD "Build Tiberian Dawn remaster dll." OFF)
option(BUILD_REMASTERRA "Build Red Alert remaster dll." OFF)
option(BUILD_VANILLATD "Build Tiberian Dawn executable." ON)
option(BUILD_VANILLARA "Build Red Alert executable." ON)
option(MAP_EDITORTD "Include internal scenario editor in Tiberian Dawn build." OFF)
option(MAP_EDITORRA "Include internal scenario editor in Red Alert build." OFF)
option(NETWORKING "Enable network play." OFF)
option(DSOUND "Enable DirectSound audio. (deprecated)" OFF)
option(DDRAW "Enable DirectDraw video backend. (deprecated)" OFF)
option(SDL2 "Enable SDL2 video backend." ON)
option(OPENAL "Enable OpenAL audio backend." ON)
option(BUILD_TESTS "Build unit tests." OFF)

add_feature_info(RemasterTD BUILD_REMASTERTD, "Remastered Tiberian Dawn dll")
add_feature_info(RemasterRA BUILD_REMASTERRA "Remastered Red Alert dll")
add_feature_info(VanillaTD BUILD_VANILLATD "Red Alert executable")
add_feature_info(VanillaRA BUILD_VANILLARA "Tiberian Dawn executable")
add_feature_info(MapEditorTD MAP_EDITORTD "Include internal scenario editor in VanillaTD")
add_feature_info(MapEditorRA MAP_EDITORRA "Include internal scenario editor in VanillaRA")
add_feature_info(Networking NETWORKING "Networking support")
add_feature_info(DirectSound DSOUND "DirectSound audio backend (deprecated)")
add_feature_info(DirectDraw DDRAW "DirectDraw video backend (deprecated)")
add_feature_info(SDL2 SDL2 "SDL2 video backend")
add_feature_info(OpenAL OPENAL "OpenAL audio backend")
add_feature_info(Tests BUILD_TESTS "Unit tests")

if(NOT BUILD_VANILLATD AND NOT BUILD_VANILLARA)
    set(DSOUND OFF)
    set(DDRAW OFF)
    set(SDL2 OFF)
    set(OPENAL OFF)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT MSVC)
    if(WIN32)
        set(CMAKE_CXX_FLAGS_DEBUG "-gstabs3")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpermissive")
        set(STATIC_LIBS "-static-libstdc++ -static-libgcc")
    else()
        set(CMAKE_CXX_FLAGS_DEBUG "-g3")
    endif()

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -fcheck-new -fsigned-char -fdata-sections -ffunction-sections -DNOMINMAX")
endif()

add_definitions(-DENGLISH -DTRUE_FALSE_DEFINED)

if(WIN32)
    add_definitions(-DWIN32 -D_WINDOWS -D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE)
    set(COMMON_LIBS winmm)
endif()

set(VANILLA_DEFS "")
set(VANILLA_LIBS "")

set(REMASTER_DEFS _USRDLL REMASTER_BUILD)
set(REMASTER_LIBS "")

find_package(ClangFormat)
include(ClangFormat)

if(BUILD_TESTS)
    # Tests need to respect some options so need to occur after the options are set.
    enable_testing()
    add_subdirectory(tests)
endif()

if(NOT BUILD_VANILLATD AND MAP_EDITORTD)
    message(WARNING "Internal scenario editor requires a Tiberian Dawn executable to be built, but it was not enabled.")
    set(BUILD_VANILLATD TRUE)
endif()

if(NOT BUILD_VANILLARA AND MAP_EDITORRA)
    message(WARNING "Internal scenario editor requires a Red Alert executable to be built, but it is not enabled.")
    set(BUILD_VANILLARA TRUE)
endif()

if(NETWORKING)
    list(APPEND VANILLA_DEFS NETWORKING)
    list(APPEND VANILLA_LIBS wsock32 ws2_32)
endif()

if(SDL2)
    find_package(SDL2 REQUIRED)
    list(APPEND VANILLA_LIBS ${SDL2_LIBRARY})
    include_directories(${SDL2_INCLUDE_DIR})
    # winuser.h defines VK_ macros but it can be prevented
    if(WIN32)
        add_definitions(-DNOVIRTUALKEYCODES)
    endif()
    set(DDRAW OFF)
endif()

if(OPENAL)
    find_package(OpenAL REQUIRED)
    list(APPEND VANILLA_LIBS OpenAL::OpenAL)
    set(DSOUND OFF)
endif()

add_subdirectory(common)
add_subdirectory(tiberiandawn)
add_subdirectory(redalert)

feature_summary(WHAT ENABLED_FEATURES DESCRIPTION "Enabled features:")
feature_summary(WHAT DISABLED_FEATURES DESCRIPTION "Disabled features:")
