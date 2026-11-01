cmake_minimum_required(VERSION 3.19)
project(pxsv-linux LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

if(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
	message(FATAL_ERROR "This repository only supports Linux")
endif()

# ---------------------------------------------------------------------------
# build_info_gen.h (git commit/branch shown in Help -> About), regenerated on every build
# ---------------------------------------------------------------------------
find_package(Git QUIET)
set(GENERATED_BUILD_INFO "${CMAKE_BINARY_DIR}/build_info_gen.h")
add_custom_target(build_info ALL
	COMMAND ${CMAKE_COMMAND}
		-DSOURCE_DIR=${CMAKE_SOURCE_DIR}
		-DGIT_EXECUTABLE=${GIT_EXECUTABLE}
		-DOUTPUT_FILE=${GENERATED_BUILD_INFO}
		-P ${CMAKE_SOURCE_DIR}/cmake/GenerateBuildInfo.cmake
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	COMMENT "Updating build info"
	VERBATIM
)

# ---------------------------------------------------------------------------
# UI toolkit (Qt6 backend)
# ---------------------------------------------------------------------------
add_subdirectory(third_party/mxtk)

# ---------------------------------------------------------------------------
# pxsv
# ---------------------------------------------------------------------------
set(PXSV_SOURCES
	src/game_shared/mathlib.cpp
	src/game_shared/matrix.cpp
	src/game_shared/stringlib.cpp
	src/utils_common/conprint.cpp
	src/utils_common/cmdlib.cpp
	src/utils_common/file_system.cpp
	src/utils_common/zone.cpp
	src/pxsv/FileAssociation.cpp
	src/pxsv/GlWindow.cpp
	src/pxsv/ViewerSettings.cpp
	src/pxsv/pakviewer.cpp
	src/pxsv/sprviewer.cpp
	src/pxsv/sprite_render.cpp
	src/pxsv/sprite_utils.cpp
)

add_executable(pxsv ${PXSV_SOURCES})
add_dependencies(pxsv build_info)

# include order matters: game_shared must come before utils_common (both have mathlib.h/stringlib.h)
target_include_directories(pxsv PRIVATE
	"src/pxsv"
	"third_party/mxtk/include"
	"third_party/mxtk/include/mx"
	"src/game_shared"
	"src/engine"
	"src/common"
	"src/public"
	"src/utils_common"
	"${CMAKE_BINARY_DIR}"
)

target_compile_definitions(pxsv PRIVATE
	IGNORE_SEARCH_IN_WADS=1
	_LINUX=1 LINUX=1
	stricmp=strcasecmp strnicmp=strncasecmp
	_snprintf=snprintf _vsnprintf=vsnprintf
)

target_compile_options(pxsv PRIVATE
	-fno-exceptions
	-fvisibility=hidden
	-Wno-write-strings
	-Wno-invalid-offsetof
	-Wno-conversion-null
)

find_package(Qt6 COMPONENTS Widgets OpenGL OpenGLWidgets REQUIRED)
target_link_libraries(pxsv PRIVATE mxtk Qt6::Widgets Qt6::OpenGL Qt6::OpenGLWidgets GL dl)

set_target_properties(pxsv PROPERTIES POSITION_INDEPENDENT_CODE ON)

install(TARGETS pxsv RUNTIME DESTINATION bin)
