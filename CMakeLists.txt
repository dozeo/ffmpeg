cmake_minimum_required(VERSION 2.8)

project(ffmpeg)

include(ExternalProject)

if(TARGET librtmp)
	set(ffmpeg_configure_flags "${ffmpeg_configure_flags} --enable-librtmp --enable-protocol=rtmp --enable-protocol=rtmps --enable-protocol=rtmpts --enable-protocol=rtmpte")
	list(APPEND dependencies "librtmp")
endif()

if(TARGET libx264)
	set(ffmpeg_configure_flags "${ffmpeg_configure_flags} --enable-libx264 --enable-encoder=libx264")
	list(APPEND dependencies "libx264")
endif()

if(MSVC)
	set(ffmpeg_configure_flags "--toolchain=msvc --target-os=win32 ${ffmpeg_configure_flags}")
	set(ffmpeg_extra_ldflags "-LIBPATH:${CMAKE_BINARY_DIR}/lib")
	set(ffmpeg_extra_cflags "-I${CMAKE_BINARY_DIR}/include")
else()
	set(ffmpeg_extra_ldflags "-L${CMAKE_BINARY_DIR}/lib")
	set(ffmpeg_extra_cflags "-I${CMAKE_BINARY_DIR}/include")
endif()

if(CMAKE_BUILD_SHARED)
	set(ffmpeg_configure_flags "--disable-static --enable-shared ${ffmpeg_configure_flags}")
else()
	set(ffmpeg_configure_flags "--disable-shared --enable-static ${ffmpeg_configure_flags}")
endif()

ExternalProject_Add(
	ffmpeg
	SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}"
	BINARY_DIR "${CMAKE_CURRENT_SOURCE_DIR}"
	CONFIGURE_COMMAND 
		bash -c
		"${CMAKE_CURRENT_SOURCE_DIR}/configure --prefix=${CMAKE_BINARY_DIR} ${ffmpeg_configure_flags} --enable-gpl --disable-doc --disable-programs --extra-cflags=${ffmpeg_extra_cflags} --extra-ldflags=${ffmpeg_extra_ldflags} --enable-protocol=tcp --enable-protocol=file"
	INSTALL_DIR "${CMAKE_BINARY_DIR}"
	DEPENDS ${dependencies}
	)

set(AVCODEC_VERSION 56)
set(AVDEVICE_VERSION 56)
set(AVFILTER_VERSION 5)
set(AVFORMAT_VERSION 56)
set(AVUTIL_VERSION 54)
set(SWSCALE_VERSION 3)
set(POSTPROC_VERSION 53)
set(SWRESAMPLE_VERSION 1)

#### Create import targets for ffmpeg libraries

file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/include)
add_library(libavcodec SHARED IMPORTED GLOBAL)
set_target_properties(libavcodec PROPERTIES
	IMPORTED_LOCATION "${CMAKE_BINARY_DIR}/bin/avcodec-${AVCODEC_VERSION}.dll"
	IMPORTED_IMPLIB "${CMAKE_BINARY_DIR}/bin/avcodec.lib"
	INTERFACE_INCLUDE_DIRECTORIES "${CMAKE_BINARY_DIR}/include"
	)