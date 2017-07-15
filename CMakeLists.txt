# Preserve backwards compatibility
cmake_minimum_required (VERSION 3.1)
set(CMAKE_CONFIGURATION_TYPES Debug RelWithDebInfo CACHE TYPE INTERNAL FORCE )

project( MinHook )
# Switch on BpcPackage
if( EXISTS "${CMAKE_CURRENT_LIST_DIR}/bpc-package/BpcPackage.cmake" )
  include( "bpc-package/BpcPackage.cmake" )
endif()

file( GLOB HDE_SOURCES "src/HDE/*.c" )
file( GLOB HDE_HEADERS "src/HDE/*.h" )

source_group( HDE FILES ${HDE_HEADERS} ${HDE_SOURCES} )

add_library( libMinHook STATIC
	include/MinHook.h
	${HDE_HEADERS}
	src/buffer.h
	src/trampoline.h
	${HDE_SOURCES}
	src/buffer.c
	src/hook.c
	src/trampoline.c
)

set_target_properties(libMinHook PROPERTIES PUBLIC_HEADER include/MinHook.h)
	
target_include_directories(libMinHook PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
  #$<INSTALL_INTERFACE:${INST_DESTINATION}/include> 
)

add_library( MinHook SHARED
	dll_resources/MinHook.rc
	dll_resources/MinHook.def	
)
target_link_libraries(MinHook PRIVATE libMinHook)


target_include_directories(MinHook PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
  #$<INSTALL_INTERFACE:${INST_DESTINATION}/include> 
)

if(CMAKE_CL_64)
	set( PLATFORM_SUFFIX x64 )
else()
	set( PLATFORM_SUFFIX x86 )
endif()

set_target_properties( libMinHook 
	PROPERTIES
		OUTPUT_NAME libMinHook.${PLATFORM_SUFFIX}
)
set_target_properties( MinHook
	PROPERTIES
		OUTPUT_NAME MinHook.${PLATFORM_SUFFIX}
)

BpcInstallPackage( NAME MinHook VERSION "1.3.3" COMPATIBILITY ExactVersion TARGETS libMinHook MinHook  )
