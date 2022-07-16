cmake_minimum_required (VERSION 3.1)
project (mman C)
set(VERSION "0.5.1")
cmake_policy(SET CMP0054 NEW)

option (BUILD_SHARED_LIBS "shared/static libs" OFF) 
option (BUILD_TESTS "tests?" OFF)

set (CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set (CMAKE_DISABLE_SOURCE_CHANGES ON)
set (CMAKE_EXPORT_COMPILE_COMMANDS ON)
set (CMAKE_VERBOSE_MAKEFILE ON)
set (CMAKE_COLOR_MAKEFILE ON)

if (NOT CMAKE_BUILD_TYPE)
	message(STATUS "No build type selected, default to Release")
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING 
                "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif (NOT CMAKE_BUILD_TYPE)

set (sources mman.c)
set (headers mman.h)
set (pkgconf ${CMAKE_CURRENT_BINARY_DIR}/mman.pc)

add_library (mman ${sources})

if (BUILD_SHARED_LIBS)
	target_compile_definitions(mman
		PUBLIC MMAN_LIBRARY_DLL
		PRIVATE MMAN_LIBRARY
	)
endif ()

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT AND DEFINED ENV{MSYSTEM_PREFIX})
	SET(CMAKE_INSTALL_PREFIX $ENV{MSYSTEM_PREFIX} CACHE PATH "Installation prefix for mingw." FORCE)
endif ()

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")


if (DEFINED ENV{MSYSTEM_PREFIX})
    # Post-build stripping 
    add_custom_command(
      TARGET mman DEPENDS mman POST_BUILD
      COMMAND $<$<CONFIG:release>:${CMAKE_STRIP}>
      ARGS --strip-unneeded $<TARGET_FILE:mman>
    )
endif ()

install(TARGETS mman
       RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
       ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
       LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )

install (FILES ${headers} DESTINATION "${INSTALL_INC_DIR}/sys")

if (DEFINED ENV{MSYSTEM_PREFIX})
    configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/mman.pc.cmakein
		${pkgconf} @ONLY)
    set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/lib/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")
    install (FILES ${pkgconf} DESTINATION "${INSTALL_PKGCONFIG_DIR}")
endif ()

if (BUILD_TESTS)
  enable_testing ()
  add_executable (t_mman test.c)
  target_link_libraries (t_mman mman)
  add_test (NAME t_mman COMMAND t_mman${CMAKE_EXECUTABLE_SUFFIX})
endif ()


