cmake_minimum_required(VERSION 3.10)

project ( libcimbar )
enable_testing()

if (MSVC)
    add_compile_options(/FI "iso646.h")
endif()

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
	set(CMAKE_INSTALL_PREFIX "${libcimbar_SOURCE_DIR}/dist" CACHE PATH "..." FORCE)
endif()

set(CMAKE_CXX_STANDARD 17)
add_definitions("-DLIBCIMBAR_PROJECT_ROOT=\"${libcimbar_SOURCE_DIR}\"")

if("${CMAKE_BUILD_TYPE}" STREQUAL "")
	set(CMAKE_BUILD_TYPE "RelWithDebInfo")
	set(CMAKE_C_FLAGS "${CFLAGS} -Wall -std=c99 -Os -fPIC")
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-Wall -Wextra -g -O2 -fPIC")
endif()

if(DEFINED USE_WASM)  # wasm build needs OPENCV_DIR defined
	set(DISABLE_TESTS true)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DGF256_TARGET_MOBILE")
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-Os")

	file( GLOB OPENCV_LIBS "${OPENCV_DIR}/opencv-build-wasm/build_wasm/lib/*.a" )
	file( GLOB opencv_include_modules "${OPENCV_DIR}/modules/*/include" )
	include_directories(
		${OPENCV_DIR}/include/
		${OPENCV_DIR}/opencv-build-wasm/build_wasm/
		${opencv_include_modules}
	)
else()  # if not wasm, go find opencv. 3 or 4 should both work
	find_package(OpenCV REQUIRED)
	include_directories(${OpenCV_INCLUDE_DIRS})
endif()

if(DEFINED BUILD_PORTABLE_LINUX)
	find_package(PkgConfig REQUIRED)
	pkg_check_modules(OPENCV4 REQUIRED opencv4)
	link_directories(${OPENCV4_STATIC_LIBRARY_DIRS})

	# statically link c++/gcc (not libc!!)
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -static-libstdc++ -static-libgcc")
endif()

if(NOT DEFINED OPENCV_LIBS)
	set(OPENCV_LIBS "opencv_calib3d" "opencv_imgcodecs" "opencv_imgproc" "opencv_photo" "opencv_core" ${OPENCV4_STATIC_LIBRARIES})
endif()

if(MSVC)
	set(CPPFILESYSTEM "")
endif()

set (PROJECTS
	src/lib/bit_file
	src/lib/chromatic_adaptation
	src/lib/cimb_translator
	src/lib/compression
	src/lib/encoder
	src/lib/fountain
	src/lib/image_hash
	src/lib/serialize
	src/lib/util

	src/third_party_lib/base91
	src/third_party_lib/cxxopts
	src/third_party_lib/intx
	src/third_party_lib/libcorrect
	src/third_party_lib/libpopcnt
	src/third_party_lib/wirehair
	src/third_party_lib/zstd
)

if(NOT DEFINED USE_WASM)
set(PROJECTS
	${PROJECTS}
	src/lib/extractor

	src/exe/cimbar
	src/exe/build_image_assets
)
endif()

include_directories(
	${libcimbar_SOURCE_DIR}/src/lib
	${libcimbar_SOURCE_DIR}/src/third_party_lib
)

foreach(proj ${PROJECTS})
	add_subdirectory(${proj} build/${proj})
endforeach()


