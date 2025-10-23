cmake_minimum_required(VERSION 3.5)

project(ElaFramework VERSION 0.1 LANGUAGES CXX)

# 基础配置

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(WIN32)
    add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")
endif()

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build Type")
endif()

# QT_SDK_DIR 配置

if(NOT DEFINED QT_SDK_DIR)
    message(WARNING "On MacOS: cmake -Bbuild -DQT_SDK_DIR=$(brew --prefix qt)")
    message(FATAL_ERROR "请在 CMake 中设置 QT_SDK_DIR 变量，指向 Qt SDK 的根路径")
endif()

message(STATUS "Qt SDK DIR: ${QT_SDK_DIR}")

SET(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/Install CACHE PATH "Installation path" FORCE)
list(APPEND CMAKE_PREFIX_PATH ${QT_SDK_DIR})

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

# RPATH

if(UNIX AND NOT APPLE)
    set(CMAKE_SKIP_INSTALL_RPATH FALSE)
    set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
    set(CMAKE_INSTALL_RPATH "${QT_SDK_DIR}/lib:${CMAKE_INSTALL_PREFIX}/ElaWidgetTools/lib")
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
    add_link_options(-Wl,--disable-new-dtags)
endif()

# 构建选项

option(BUILD_ELAPACKETIO "Build ElaPacketIO" OFF)
option(BUILD_EXAMPLE "Build ElaWidgetToolsExample" OFF)

# 子模块构建

add_subdirectory(ElaWidgetTools)

if (WIN32 AND BUILD_ELAPACKETIO)
    SET(QT_SDK_DIR D:/Qt/5.15.2/msvc2019_64 CACHE PATH "QT SDK DIR" FORCE)
    add_definitions(-DBUILD_WITH_ELAPACKETIO)
    add_subdirectory(ElaPacketIO)
endif ()

if(BUILD_EXAMPLE)
    add_subdirectory(ElaWidgetToolsExample)
endif()

# Mac 特殊处理

if (${QT_VERSION} VERSION_LESS 6.1.0)
    set(BUNDLE_ID_OPTION MACOSX_BUNDLE_GUI_IDENTIFIER com.example.ElaWidgetTools)
endif ()
