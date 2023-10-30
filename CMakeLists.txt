cmake_minimum_required(VERSION 3.24)

include(FetchContent)

set(TARGET detours)
project(${TARGET} VERSION 1.0)

option(DETOURS_CL_17_OR_NEWER "DETOURS_CL_17_OR_NEWER" ON)
option(DETOURS_WIN_7 "DETOURS_WIN_7" OFF)
option (DETOURS_SOURCE_BROWSING "DETOURS_SOURCE_BROWSING" ON)

FetchContent_Declare(
	libdetours
	GIT_REPOSITORY https://github.com/microsoft/Detours.git
	GIT_TAG        e4bfd6b03e50de46b47abfbd1e46b384f0c5f833
)

FetchContent_MakeAvailable(libdetours)
set(DETOURS_SOURCE "${libdetours_SOURCE_DIR}/src")

file(GLOB_RECURSE srcs "${DETOURS_SOURCE}/*.cpp" "${DETOURS_SOURCE}/*.h")
add_library(${TARGET} STATIC ${srcs})

set_property(
    SOURCE ${DETOURS_SOURCE}/uimports.cpp
    APPEND PROPERTY HEADER_FILE_ONLY true
)

set(COMPILE_OPTIONS /nologo /W4 /WX /we4777 /we4800 /Zi /MT /Gy /Gm- /Zl /Od)

if ((NOT ${DETOURS_WIN_7}) AND ${DETOURS_CL_17_OR_NEWER})
	list(APPEND COMPILE_OPTIONS /D_USING_V110_SDK71_)
endif()

if(DETOURS_SOURCE_BROWSING)
	list(APPEND COMPILE_OPTIONS /FR)
else()
	list(APPEND COMPILE_OPTIONS /DWIN32_LEAN_AND_MEAN /D_WIN32_WINNT=0x501)
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	add_definitions(-D_AMD64_)
else()
	add_definitions(-D_X86_)
endif()

include(CMakePrintHelpers)
cmake_print_variables(COMPILE_OPTIONS)

target_compile_options(${TARGET} PRIVATE ${COMPILE_OPTIONS})
target_include_directories(${TARGET} PUBLIC ${DETOURS_SOURCE})