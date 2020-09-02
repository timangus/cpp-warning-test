cmake_minimum_required(VERSION 3.14)

project(cpp-warning-test LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wpedantic -Wall -Wextra -Wcast-align -Wcast-qual \
        -Wdisabled-optimization -Wformat=2 -Winit-self \
        -Wmissing-include-dirs -Wold-style-cast -Woverloaded-virtual \
        -Wnon-virtual-dtor -Wredundant-decls -Wshadow")
endif()

if(MSVC)
    add_definitions(-DUNICODE -D_UNICODE)

    # MSVC is picky with UTF8 files recognition so force it on
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /utf-8")
    # Disable large numbers of encoding warnings for boost with utf8 encoding on
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4828")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4250 /wd4996")

    set(VC_ANALYZE_FLAGS "/analyze /analyze:ruleset \
        ${CMAKE_SOURCE_DIR}\\scripts\\VCAnalyze.ruleset")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${VC_ANALYZE_FLAGS}")

    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} \
        /DYNAMICBASE /NXCOMPAT /MAP /debug")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} \
        /MAP /debug")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} \
        /permissive- \
        /Zc:rvalueCast /Zc:inline /Zc:strictStrings \
        /Zc:wchar_t /Zc:throwingNew")
endif()

set(CMAKE_CXX_CLANG_TIDY clang-tidy -checks=-*,readability-*)
set(CMAKE_CXX_CPPCHECK cppcheck --enable=all)

# QtCreator supports the following variables for Android, which are identical to qmake Android variables.
# Check http://doc.qt.io/qt-5/deployment-android.html for more information.
# They need to be set before the find_package(Qt5 ...) call.

#if(ANDROID)
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
#    if (ANDROID_ABI STREQUAL "armeabi-v7a")
#        set(ANDROID_EXTRA_LIBS
#            ${CMAKE_CURRENT_SOURCE_DIR}/path/to/libcrypto.so
#            ${CMAKE_CURRENT_SOURCE_DIR}/path/to/libssl.so)
#    endif()
#endif()

find_package(QT NAMES Qt6 Qt5 COMPONENTS Core Quick REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core Quick REQUIRED)

if(ANDROID)
    add_library(cpp-warning-test SHARED
      main.cpp
      qml.qrc
    )
else()
    add_executable(cpp-warning-test
      main.cpp
      qml.qrc
    )
endif()

target_compile_definitions(cpp-warning-test
  PRIVATE $<$<OR:$<CONFIG:Debug>,$<CONFIG:RelWithDebInfo>>:QT_QML_DEBUG>)
target_link_libraries(cpp-warning-test
  PRIVATE Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Quick)
