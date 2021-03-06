cmake_minimum_required(VERSION 3.0.0)
# required for finding Python 3.4.
# (determined via git tag --contains)

# main build configuration file

# text art: figlet -f rounded "[SFT] openage" | sed -e 's/\\/\\\\/g'
message("(running cmake...)

 ___  ______ _______ _______ ___
|  _)/ _____|_______|_______|_  |
| | ( (____  _____      _     | |    ___  ____  _____ ____  _____  ____ _____
| |  \\____ \\|  ___)    | |    | |   / _ \\|  _ \\| ___ |  _ \\(____ |/ _  | ___ |
| |_ _____) ) |        | |   _| |  | |_| | |_| | ____| | | / ___ ( (_| | ____|
|___|______/|_|        |_|  (___|   \\___/|  __/|_____)_| |_\\_____|\\___ |_____)
                                         |_|                     (_____|

Welcome to the SFT technologies computer-aided openage build system!

You have chosen, or been chosen, to attempt the daring task of building openage.
If you have installed all the dependencies that are conveniently listed in
[doc/building.md], this _might_ just work!

If it doesn't, consider reporting the issue/asking for help in #sfttech on freenode.net.
")

project(openage C CXX)

# options: keep up to date with those in ./configure!
if(NOT DEFINED WANT_INOTIFY)
	set(WANT_INOTIFY if_available)
endif()

if(NOT DEFINED WANT_GPERFTOOLS_PROFILER)
	set(WANT_GPERFTOOLS_PROFILER if_available)
endif()

if(NOT DEFINED WANT_GPERFTOOLS_TCMALLOC)
	set(WANT_GPERFTOOLS_TCMALLOC false)
endif()

set(BUILDSYSTEM_DIR "${CMAKE_SOURCE_DIR}/buildsystem")
set(CMAKE_MODULE_PATH "${BUILDSYSTEM_DIR}/modules/")

# include build configuration modules
include("CTest")

include("${BUILDSYSTEM_DIR}/util.cmake")

include("${BUILDSYSTEM_DIR}/cpp.cmake")
include("${BUILDSYSTEM_DIR}/python.cmake")
include("${BUILDSYSTEM_DIR}/codegen.cmake")

include("${BUILDSYSTEM_DIR}/assets.cmake")
include("${BUILDSYSTEM_DIR}/version.cmake")

include("${BUILDSYSTEM_DIR}/doxygen.cmake")

disallow_in_source_builds()

# create documentation
doxygen_configure(libopenage/ openage/)

add_subdirectory(libopenage/)
set(PYEXT_LINK_LIBRARY libopenage)
set(PYEXT_CXXFLAGS "${PYEXT_CXXFLAGS} -include \"${CMAKE_SOURCE_DIR}/libopenage/pyinterface/hacks.h\"")
add_cython_modules(EMBED NOINSTALL run.py)
add_subdirectory(openage/)
add_subdirectory(dist/)

python_finalize()

message("")

print_config_options()

# show build configuration overview
message("${PROJECT_NAME} ${PROJECT_VERSION}

         compiler | ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}
         cxxflags | ${CMAKE_CXX_FLAGS}
        build dir | ${CMAKE_BINARY_DIR}
   install prefix | ${CMAKE_INSTALL_PREFIX}
py install prefix | ${CMAKE_PY_INSTALL_PREFIX}
")

# detect missing run-time dependencies
warn_runtime_depends()

# don't print 'Built target ...' messages
# this will hopefully be merged into CMake 3.3; until then, this will have
# no effect.
# http://www.cmake.org/pipermail/cmake-developers/2015-July/025649.html
set_property(GLOBAL PROPERTY TARGET_MESSAGES OFF)
