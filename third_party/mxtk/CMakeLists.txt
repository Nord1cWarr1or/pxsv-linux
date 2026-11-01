# mxtk - small widget toolkit by Mete Ciragan, Qt backend only (Linux build)
cmake_minimum_required(VERSION 3.19)
project(mxtk)

set(CMAKE_AUTOMOC ON)

file(GLOB MXTK_SOURCES "src/common/*.cpp" "src/qt/*.cpp")
add_library(mxtk STATIC ${MXTK_SOURCES})

target_include_directories(mxtk PRIVATE
	"include"
	"src/qt"
)

target_compile_options(mxtk PRIVATE -fno-exceptions -Wno-write-strings -fvisibility=hidden)
target_compile_definitions(mxtk PRIVATE _LINUX=1 LINUX=1)

find_package(Qt6 COMPONENTS Widgets OpenGL OpenGLWidgets REQUIRED)
target_link_libraries(mxtk PRIVATE
	Qt6::Widgets
	Qt6::OpenGL
	Qt6::OpenGLWidgets
	GL
)

set_target_properties(mxtk PROPERTIES POSITION_INDEPENDENT_CODE ON)
