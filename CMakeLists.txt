# Preserve backwards compatibility
cmake_minimum_required (VERSION 3.1)
set(CMAKE_CONFIGURATION_TYPES Debug RelWithDebInfo CACHE TYPE INTERNAL FORCE )

project (MinHook)

# Last is ai-master versions counted up each time we put it in the libraries again
set( PACKAGE_VERSION 1.3.3 )
set( PACK_FOR_ALL_COMPILERS True )

# BEGIN EXPORT AND INSTALL RELATED STUFF -- SHOULD GO TO MACRO FOR OUR PURPOSES

string( TOLOWER ${CMAKE_PROJECT_NAME} package )
string( TOUPPER ${CMAKE_PROJECT_NAME} PACKAGE )

if( PACK_FOR_ALL_COMPILERS )
	# Version for installation to ALL_COMPIERS
	if( CMAKE_CL_64 )
		set( PLATFORM x64 )
	else()
		set( PLATFORM win32 )
	endif()

	set( CONF_ROOT "cmake/ALL_COMPILERS/${PLATFORM}" )
	set( INST_ROOT "ALL_COMPILERS" )
else()
	# Version for compiler specific packages
	if (CMAKE_CL_64)
 		set( BPC_COMPILER MSVC-64-14.0 )
	else()
		set( BPC_COMPILER MSVC-32-14.0 )
	endif()
	set( CONF_ROOT "cmake/${BPC_COMPILER}" )
	set( INST_ROOT ${BPC_COMPILER} )
endif()

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set (CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}-install/" CACHE PATH "CMake install Directory" FORCE )
endif()

# END EXPORT AND INSTALL RELATED STUFF

set( CONF_DESTINATION "${CONF_ROOT}/Minhook-${PACKAGE_VERSION}" )
set( INST_DESTINATION "${INST_ROOT}/${package}/${package}-${PACKAGE_VERSION}" )

foreach( config ${CMAKE_CONFIGURATION_TYPES} )
	string( TOUPPER ${config} CONFIG )
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${CONFIG} "${CMAKE_BINARY_DIR}/bin/${config}")
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${CONFIG} "${CMAKE_BINARY_DIR}/bin/${config}")
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${CONFIG} "${CMAKE_BINARY_DIR}/lib/${config}")
endforeach()

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
  $<INSTALL_INTERFACE:${INST_DESTINATION}/include> 
)

add_library( MinHook SHARED
	dll_resources/MinHook.rc
	dll_resources/MinHook.def	
)
target_link_libraries(MinHook PRIVATE libMinHook)


target_include_directories(MinHook PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
  $<INSTALL_INTERFACE:${INST_DESTINATION}/include> 
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

# We may want to use ExactVersion here to make sure old builds will be EXACTLY the same when reproduced
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/Minhook-config-version.cmake"
	VERSION ${PACKAGE_VERSION}
	COMPATIBILITY ExactVersion
)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/Minhook-config-version.cmake" 
        DESTINATION ${CONF_DESTINATION} )

foreach( config ${CMAKE_CONFIGURATION_TYPES} )
	string( TOUPPER ${config} CONFIG )
	install( TARGETS libMinHook MinHook EXPORT MinHook.${config} CONFIGURATIONS ${config} 
		RUNTIME DESTINATION ${INST_DESTINATION}/bin/${config}
		LIBRARY DESTINATION ${INST_DESTINATION}/bin/${config}
		ARCHIVE DESTINATION ${INST_DESTINATION}/lib/${config}
		PUBLIC_HEADER DESTINATION ${INST_DESTINATION}/include
		INCLUDES DESTINATION ${INST_DESTINATION}/include
	)
	install(EXPORT MinHook.${config} DESTINATION ${CONF_DESTINATION} CONFIGURATIONS ${config} FILE ${package}-config.cmake)
endforeach()
