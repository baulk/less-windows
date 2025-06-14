cmake_minimum_required(VERSION 3.20)

project(less)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR AND NOT MSVC_IDE)
  message(
    FATAL_ERROR
      "In-source builds are not allowed.
CMake would overwrite the makefiles distributed with LESS.
Please create a directory and run cmake from there, passing the path
to this source directory as the last argument.
This process created the file `CMakeCache.txt' and the directory `CMakeFiles'.
Please delete them.")
endif()

if(MSVC)
  set(CMAKE_MSVC_RUNTIME_LIBRARY_DEFAULT "")
  # FIXME: This block should go away after a transition period.
  if(NOT CMAKE_VERSION VERSION_LESS 3.15)
    # Filter out MSVC runtime library flags that may have come from the cache of an existing build tree or from scripts.
    foreach(l C CXX)
      foreach(c DEBUG MINSIZEREL RELEASE RELWITHDEBINFO)
        string(REGEX REPLACE "[-/]M[DT]d?( |$)" "" "CMAKE_${l}_FLAGS_${c}" "${CMAKE_${l}_FLAGS_${c}}")
      endforeach()
    endforeach()
  endif()
  # http://stackoverflow.com/a/32128977/486990
  add_compile_options(
    "$<$<CONFIG:Debug>:/MTd>" # -MTd
    "$<$<CONFIG:RelWithDebInfo>:/MT>" # -MT
    "$<$<CONFIG:Release>:/MT>" # * MT
    "$<$<CONFIG:MinSizeRel>:/MT>")
  add_compile_options("-D_CRT_SECURE_NO_WARNINGS=1")
  add_compile_options("-utf-8")
  add_compile_options("-permissive-")
  add_compile_options("-Zc:__cplusplus")
  # add_compile_options("-DUNICODE=1")
  # add_compile_options("-D_UNICODE=1")
  add_compile_options("-W3")
  set(CMAKE_RC_FLAGS "${CMAKE_RC_FLAGS} -c65001")
else()
  # add_compile_options("-DUNICODE=1")
  # add_compile_options("-D_UNICODE=1")
  add_compile_options("-g")
  add_compile_options("-Wall")
  add_compile_options("-Wextra")
  if(USE_SANITIZE)
    add_compile_options("-fsanitize=address")
    add_compile_options("-fno-omit-frame-pointer")
    set(CMAKE_EXE_LINKER_FLAGS "-fsanitize=address ")
  endif()
endif()

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)


set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
# Gen version
include(VersionFromVCS)
get_source_info(${CMAKE_CURRENT_SOURCE_DIR} LESS_REVISION LESS_REMOTE_URL LESS_REFNAME)
string(TIMESTAMP LESS_COPYRIGHT_YEAR "%Y")

# LESS_REFNAME maybe defined by ci
if("${LESS_REFNAME}" STREQUAL "none")
  if(DEFINED ENV{LESS_REFNAME})
    set(LESS_REFNAME $ENV{LESS_REFNAME})
  endif()
endif()

if("${LESS_REFNAME}" MATCHES "refs/tags/*")
  string(TIMESTAMP LESS_BUILD_TIME "%Y-%m-%dT%H:%M:%SZ")
else()
  set(LESS_BUILD_TIME "none")
endif()

if(DEFINED ENV{GITHUB_RUN_NUMBER})
  set(LESS_VERSION_BUILD $ENV{GITHUB_RUN_NUMBER})
  message(STATUS "UNRAR detect GITHUB_RUN_NUMBER: $ENV{GITHUB_RUN_NUMBER}")
else()
  set(LESS_VERSION_BUILD 1)
endif()

set(LESS_VERSION_MAJOR 679)
set(LESS_VERSION_MINOR 0)
set(LESS_VERSION_PATCH 0)
set(PACKAGE_VERSION "${LESS_VERSION_MAJOR}")

string(TOLOWER "${CMAKE_C_COMPILER_ARCHITECTURE_ID}" LESS_ARCH_NAME)
set(CPACK_SYSTEM_NAME "${LESS_ARCH_NAME}")
set(CPACK_PACKAGE_NAME "less")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "less")
set(CPACK_PACKAGE_VERSION_MAJOR ${LESS_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${LESS_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${LESS_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION ${PACKAGE_VERSION})
set(CPACK_PACKAGE_VENDOR "less")
set(CPACK_PACKAGE_CONTACT "Mark Nudelman")
include(CPack)

include_directories("./less")

add_executable(less
  "less/main.c"
  "less/screen.c"
  "less/brac.c"
  "less/ch.c"
  "less/charset.c"
  "less/cmdbuf.c"
  "less/command.c"
  "less/cvt.c"
  "less/decode.c"
  "less/edit.c"
  "less/evar.c"
  "less/filename.c"
  "less/forwback.c"
  "less/help.c"
  "less/ifile.c"
  "less/input.c"
  "less/jump.c" 
  "less/lesskey_parse.c"
  "less/line.c"
  "less/linenum.c"
  "less/lsystem.c"
  "less/mark.c"
  "less/optfunc.c"
  "less/option.c"
  "less/opttbl.c"
  "less/os.c"
  "less/output.c"
  "less/pattern.c"
  "less/position.c"
  "less/prompt.c"
  "less/search.c"
  "less/signal.c"
  "less/tags.c"
  "less/ttyin.c"
  "less/version.c"
  "less/xbuf.c"
  "less/regexp.c"
)

add_executable(lesskey
  "less/lesskey.c"
  "less/lesskey_parse.c"
  "less/version.c"
  "less/xbuf.c"
)

install(TARGETS less DESTINATION .)
install(TARGETS lesskey DESTINATION .)
install(FILES LICENSE DESTINATION .)
