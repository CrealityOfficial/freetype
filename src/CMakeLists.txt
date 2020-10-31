include(CheckIncludeFile)

if (APPLE)
  if (DEFINED IOS_PLATFORM)
    if (NOT "${IOS_PLATFORM}" STREQUAL "OS"
        AND NOT "${IOS_PLATFORM}" STREQUAL "SIMULATOR")
      message(FATAL_ERROR
        "IOS_PLATFORM must be set to either OS or SIMULATOR")
    endif ()
    if (NOT "${CMAKE_GENERATOR}" STREQUAL "Xcode")
      message(AUTHOR_WARNING
        "You should use Xcode generator with IOS_PLATFORM enabled to get Universal builds.")
    endif ()

    set(CMAKE_TOOLCHAIN_FILE
      ${CMAKE_SOURCE_DIR}/builds/cmake/iOS.cmake)
  endif ()
else ()
  if (DEFINED IOS_PLATFORM)
    message(FATAL_ERROR "IOS_PLATFORM is not supported on this platform")
  endif ()
endif ()

# Add local cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/builds/cmake)

set(VERSION_MAJOR "2")
set(VERSION_MINOR "9")
set(VERSION_PATCH "0")

set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})
set(SHARED_LIBRARY_VERSION ${VERSION_MAJOR}.${VERSION_MINOR})

# Compiler definitions for building the library
add_definitions(-DFT2_BUILD_LIBRARY)

# Find dependencies
foreach (d ZLIB BZip2 PNG HarfBuzz)
	string(TOUPPER "${d}" D)
	
	if (DEFINED WITH_${d} OR DEFINED WITH_${D})
	if (WITH_${d} OR WITH_${D})
		find_package(${d} QUIET REQUIRED)
	endif ()
	else ()
	find_package(${d} QUIET)
	endif ()
	
	if (${d}_FOUND OR ${D}_FOUND)
	message(STATUS "Building with ${d}")
	endif ()
endforeach ()

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config")

# Create the configuration file
message(STATUS
  "Creating file ${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h")

if (UNIX)
  check_include_file("unistd.h" HAVE_UNISTD_H)
  check_include_file("fcntl.h" HAVE_FCNTL_H)
  check_include_file("stdint.h" HAVE_STDINT_H)

  file(READ "${CMAKE_CURRENT_SOURCE_DIR}/builds/unix/ftconfig.in"
    FTCONFIG_H)
  if (HAVE_UNISTD_H)
    string(REGEX REPLACE
      "#undef +(HAVE_UNISTD_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  if (HAVE_FCNTL_H)
    string(REGEX REPLACE
      "#undef +(HAVE_FCNTL_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  if (HAVE_STDINT_H)
    string(REGEX REPLACE
      "#undef +(HAVE_STDINT_H)" "#define \\1"
      FTCONFIG_H "${FTCONFIG_H}")
  endif ()
  string(REPLACE "/undef " "#undef "
    FTCONFIG_H "${FTCONFIG_H}")
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h-new"
    "${FTCONFIG_H}")
else ()
  file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/freetype/config/ftconfig.h"
    FTCONFIG_H)
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h-new"
    "${FTCONFIG_H}")
endif ()
execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different
  "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h-new"
  "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h")

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/freetype/config/ftoption.h"
  FTOPTION_H)
if (ZLIB_FOUND)
  string(REGEX REPLACE
    "/\\* +(#define +FT_CONFIG_OPTION_SYSTEM_ZLIB) +\\*/" "\\1"
    FTOPTION_H "${FTOPTION_H}")
endif ()
if (BZIP2_FOUND)
  string(REGEX REPLACE
    "/\\* +(#define +FT_CONFIG_OPTION_USE_BZIP2) +\\*/" "\\1"
    FTOPTION_H "${FTOPTION_H}")
endif ()
if (PNG_FOUND)
  string(REGEX REPLACE
    "/\\* +(#define +FT_CONFIG_OPTION_USE_PNG) +\\*/" "\\1"
    FTOPTION_H "${FTOPTION_H}")
endif ()
if (HARFBUZZ_FOUND)
  string(REGEX REPLACE
    "/\\* +(#define +FT_CONFIG_OPTION_USE_HARFBUZZ) +\\*/" "\\1"
    FTOPTION_H "${FTOPTION_H}")
endif ()
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h-new"
  "${FTOPTION_H}")
execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different
  "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h-new"
  "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h")


# Specify library include directories
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")
include_directories(BEFORE "${CMAKE_CURRENT_BINARY_DIR}/include")


file(GLOB PUBLIC_HEADERS "include/ft2build.h" "include/freetype/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "include/freetype/config/*.h")
file(GLOB PRIVATE_HEADERS "include/freetype/internal/*.h")


set(BASE_SRCS
  src/autofit/autofit.c
  src/base/ftbase.c
  src/base/ftbbox.c
  src/base/ftbdf.c
  src/base/ftbitmap.c
  src/base/ftcid.c
  src/base/ftfntfmt.c
  src/base/ftfstype.c
  src/base/ftgasp.c
  src/base/ftglyph.c
  src/base/ftgxval.c
  src/base/ftinit.c
  src/base/ftlcdfil.c
  src/base/ftmm.c
  src/base/ftotval.c
  src/base/ftpatent.c
  src/base/ftpfr.c
  src/base/ftstroke.c
  src/base/ftsynth.c
  src/base/ftsystem.c
  src/base/fttype1.c
  src/base/ftwinfnt.c
  src/bdf/bdf.c
  src/bzip2/ftbzip2.c
  src/cache/ftcache.c
  src/cff/cff.c
  src/cid/type1cid.c
  src/gzip/ftgzip.c
  src/lzw/ftlzw.c
  src/pcf/pcf.c
  src/pfr/pfr.c
  src/psaux/psaux.c
  src/pshinter/pshinter.c
  src/psnames/psnames.c
  src/raster/raster.c
  src/sfnt/sfnt.c
  src/smooth/smooth.c
  src/truetype/truetype.c
  src/type1/type1.c
  src/type42/type42.c
  src/winfonts/winfnt.c
)

if (WIN32)
  set(BASE_SRCS ${BASE_SRCS} builds/windows/ftdebug.c)
elseif (WINCE)
  set(BASE_SRCS ${BASE_SRCS} builds/wince/ftdebug.c)
else ()
  set(BASE_SRCS ${BASE_SRCS} src/base/ftdebug.c)
endif ()

set(Defs freetype_EXPORTS)
__add_real_target(freetype dll 
		SOURCE ${PUBLIC_HEADERS} ${PUBLIC_CONFIG_HEADERS} ${PRIVATE_HEADERS} ${BASE_SRCS}
		DEF ${Defs})

if (NOT CMAKE_VERSION VERSION_LESS 2.8.12)
  target_include_directories(freetype
		PUBLIC $<INSTALL_INTERFACE:include/freetype2>)
endif ()

if (CMAKE_VERSION VERSION_LESS 2.8.12)
  set(MAYBE_PRIVATE "")
else ()
  set(MAYBE_PRIVATE "PRIVATE")
endif ()

if (ZLIB_FOUND)
  target_link_libraries(freetype ${MAYBE_PRIVATE} ${ZLIB_LIBRARIES})
  include_directories(${ZLIB_INCLUDE_DIRS})
endif ()
if (BZIP2_FOUND)
  target_link_libraries(freetype ${MAYBE_PRIVATE} ${BZIP2_LIBRARIES})
  include_directories(${BZIP2_INCLUDE_DIR}) # not BZIP2_INCLUDE_DIRS
endif ()
if (PNG_FOUND)
  add_definitions(${PNG_DEFINITIONS})
  target_link_libraries(freetype ${MAYBE_PRIVATE} ${PNG_LIBRARIES})
  include_directories(${PNG_INCLUDE_DIRS})
endif ()
if (HARFBUZZ_FOUND)
  target_link_libraries(freetype ${MAYBE_PRIVATE} ${HARFBUZZ_LIBRARIES})
  include_directories(${HARFBUZZ_INCLUDE_DIRS})
endif ()


# Installations
# Note the trailing slash in the argument to the `DIRECTORY' directive
  install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION include/freetype2
    PATTERN "internal" EXCLUDE
    PATTERN "ftconfig.h" EXCLUDE
    PATTERN "ftoption.h" EXCLUDE
    )
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h
    DESTINATION include/freetype2/freetype/config
    )

set_property(TARGET freetype PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include/)