project(mldr)

cmake_minimum_required(VERSION 3.10)

include_directories(include)

set_source_files_properties(${CMAKE_BINARY_DIR}/src/external/darlingserver/src/rpc.c PROPERTIES
	GENERATED TRUE
)

add_library(mldr_dserver_rpc ${CMAKE_BINARY_DIR}/src/external/darlingserver/src/rpc.c)
add_library(mldr32_dserver_rpc ${CMAKE_BINARY_DIR}/src/external/darlingserver/src/rpc.c)

add_dependencies(mldr_dserver_rpc generate_dserver_rpc_wrappers)
add_dependencies(mldr32_dserver_rpc generate_dserver_rpc_wrappers)

target_compile_options(mldr_dserver_rpc PRIVATE -include ${CMAKE_CURRENT_SOURCE_DIR}/resources/dserver-rpc-defs.h)
target_compile_options(mldr32_dserver_rpc PRIVATE -include ${CMAKE_CURRENT_SOURCE_DIR}/resources/dserver-rpc-defs.h)

target_compile_options(mldr32_dserver_rpc PRIVATE -m32)
target_link_options(mldr32_dserver_rpc PRIVATE -m32)

set(mldr_sources
	mldr.c
	commpage.c
	elfcalls/elfcalls.c
	elfcalls/threads.c
)

add_executable(mldr ${mldr_sources})

add_executable(mldr32 ${mldr_sources})
target_compile_options(mldr32 PRIVATE -m32)
target_link_options(mldr32 PRIVATE -m32)

target_compile_options(mldr PRIVATE -pthread)
target_compile_options(mldr32 PRIVATE -pthread)
target_link_options(mldr PRIVATE -pthread)
target_link_options(mldr32 PRIVATE -pthread)

target_link_libraries(mldr PRIVATE -lrt -ldl mldr_dserver_rpc)
target_link_libraries(mldr32 PRIVATE -lrt -ldl mldr32_dserver_rpc)

install(TARGETS mldr mldr32 DESTINATION libexec/darling/usr/libexec/darling)

include(setcap)
setcap(libexec/darling/usr/libexec/darling/mldr cap_sys_rawio,cap_sys_resource+ep)
setcap(libexec/darling/usr/libexec/darling/mldr32 cap_sys_rawio,cap_sys_resource+ep)

configure_file(darling.conf.in darling.conf @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/darling.conf" DESTINATION lib/binfmt.d)