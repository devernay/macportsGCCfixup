cmake_minimum_required(VERSION 2.8)
project (install-mp-gcc)
set (GCC_VERSION "4.8.5" CACHE STRING  "GCC version to build the driver for")
string (REGEX MATCHALL "[0-9]+" GCC_VERSION_COMPONENTS ${GCC_VERSION})
list (GET GCC_VERSION_COMPONENTS 0 GCC_VERSION_MAJOR)
list (GET GCC_VERSION_COMPONENTS 1 GCC_VERSION_MINOR)
list (GET GCC_VERSION_COMPONENTS 2 GCC_VERSION_MICRO)
set (install-mp-gcc_VERSION_MAJOR 1)
set (install-mp-gcc_VERSION_MINOR 0)
set (install-mp-gcc_VERSION_MICRO 0)

CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/config.h.in ${PROJECT_BINARY_DIR}/config.h)
# add the binary tree directory to the search path for 
# include files
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )

if( NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin" )
    MESSAGE( FATAL_ERROR
             "This package is meant to further enhance the macports version of GCC.  As such it is only run on an OSX system." )
endif()

EXECUTE_PROCESS(COMMAND uname -r
                COMMAND awk -F "."  {print$1} 
                OUTPUT_VARIABLE OSVERSION 
                OUTPUT_STRIP_TRAILING_WHITESPACE)

find_path( PORTS_PATH bin/port
           /opt/local )

set (CMAKE_INSTALL_PREFIX "${PORTS_PATH}")

if( NOT PORTS_PATH )
    MESSAGE( FATAL_ERROR
             "It appears that macports has not yet been installed.  You can find a installable package for macports at http://www.macports.org/install.php ." )
endif()

find_program ( GCC
	       NAMES gcc-mp-${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR} gcc-mp-${GCC_VERSION_MAJOR} gcc-${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR} gcc-${GCC_VERSION_MAJOR}
	       PATHS ${PORTS_PATH}/bin )

find_program ( GXX
	       NAMES g++-mp-${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR} g++-mp-${GCC_VERSION_MAJOR} g++-${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR} g++-${GCC_VERSION_MAJOR}
	       PATHS ${PORTS_PATH}/bin )

execute_process (
  COMMAND         ${GCC} -dumpmachine
  OUTPUT_VARIABLE GCC_SYSTEM_NAME
)
string(STRIP ${GCC_SYSTEM_NAME} GCC_SYSTEM_NAME)

execute_process (
  COMMAND         ${GXX} -dumpmachine
  OUTPUT_VARIABLE GXX_SYSTEM_NAME
)
string(STRIP ${GXX_SYSTEM_NAME} GXX_SYSTEM_NAME)

get_filename_component(GCCNAME ${GCC} NAME)
get_filename_component(GXXNAME ${GXX} NAME)
get_filename_component(GCCDIR ${GCC} DIRECTORY)
get_filename_component(GXXDIR ${GXX} DIRECTORY)

find_program ( GCC_FULL
	       NAMES ${GCC_SYSTEM_NAME}-${GCCNAME}
	       PATHS ${GCCDIR} )

find_program ( GXX_FULL
	       NAMES ${GXX_SYSTEM_NAME}-${GXXNAME}
	       PATHS ${GXXDIR} )

get_filename_component(GCC_FULL_NAME ${GCC_FULL} NAME)
get_filename_component(GXX_FULL_NAME ${GXX_FULL} NAME)
string(REPLACE "x86_64" "" GCC_FULL_NAME_NOARCH ${GCC_FULL_NAME})
string(REPLACE "x86_64" "" GXX_FULL_NAME_NOARCH ${GXX_FULL_NAME})

if( NOT GCC )
     MESSAGE( FATAL_ERROR
              "Unable to find the macports version of GCC, please install it by running \"${PORTS_PATH}/bin/port install gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR} +universal\"." )
endif()

if( NOT GCC_FULL )
     MESSAGE( FATAL_ERROR
              "Unable to find the full gcc ${GCC_SYSTEM_NAME}-${GCCNAME} in ${GCCDIR}." )
endif()

find_path( XCODE_PLUGIN_PATH GCC\ 4.2.xcplugin
           /Developer/Library/Xcode/PrivatePlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins
           /Developer/Library/Xcode/Plug-ins
           /Applications/Xcode.app/Contents/PlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins )

if( NOT XCODE_PLUGIN_PATH )
     MESSAGE( "WARNING: Unable to find xcode xcplugin path, MacPorts GCC will not be integrated with Xcode.  Commandline multi arch builds will still be possible" )
endif()

ADD_EXECUTABLE( ${GCCNAME} driverdriver.c )
ADD_EXECUTABLE( ${GXXNAME} driverdriver.c )

SET_TARGET_PROPERTIES(  ${GCCNAME}
                        PROPERTIES
                        COMPILE_FLAGS "-DPDN='\"${GCC_FULL_NAME_NOARCH}\"'" )

SET_TARGET_PROPERTIES(  ${GXXNAME}
                        PROPERTIES
                        COMPILE_FLAGS "-DPDN='\"${GXX_FULL_NAME_NOARCH}\"'" )

find_path( DARWIN_H_INCLUDE_PATH darwin.h
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/i386/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/i386/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/i386/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/i386/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/ )

if( DARWIN_H_INCLUDE_PATH )
    MESSAGE( "Found darwin.h at ${DARWIN_H_INCLUDE_PATH}" )
    include_directories( ${DARWIN_H_INCLUDE_PATH} )
else()
    MESSAGE( "${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/config/i386" )
    MESSAGE( FATAL_ERROR "Unable to find darwin.h.  Find darwin.h and add the path to the DARWIN_H_INCLUDE_PATH search path list in CMakeLists.txt" )
endif()


find_path( LIBIBERTY_INCLUDES libiberty.h
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/i386-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/
           ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}/gcc/x86_64-apple-darwin${OSVERSION}/${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.${GCC_VERSION_MICRO}/plugin/include/ )

if( LIBIBERTY_INCLUDES )
    MESSAGE( "Found libiberty includes at ${LIBIBERTY_INCLUDES}" )
    include_directories( ${LIBIBERTY_INCLUDES} )
else()
    MESSAGE( FATAL_ERROR "Unable to find libiberty includes.  Find libiberty.h and add the path to the LIBIBERTY_INCLUDES search path list in CMakeLists.txt" )
endif()

find_library( IBERTY_LIB 
              NAMES libiberty.a
              PATHS ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/x86_64/ ${PORTS_PATH}/lib/gcc${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR}/ )

if( IBERTY_LIB )
    MESSAGE( "Found libiberty at ${IBERTY_LIB}" )
else()
    MESSAGE( FATAL_ERROR "Unable to find libiberty.  Find libiberty.a in the gcc macports directory and add the path to the IBERTY_LIB search path list in CMakeLists.txt" )
endif()

TARGET_LINK_LIBRARIES( ${GCCNAME} ${IBERTY_LIB} )
TARGET_LINK_LIBRARIES( ${GXXNAME} ${IBERTY_LIB} )

# do not install these, they should be useless
#CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/i386-gcc-mp-version.in ${PROJECT_BINARY_DIR}/i386-${GCCNAME})
#CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/i386-g++-mp-version.in ${PROJECT_BINARY_DIR}/i386-${GXXNAME})
#CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/x86_64-gcc-mp-version.in ${PROJECT_BINARY_DIR}/x86_64-${GCCNAME})
#CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/x86_64-g++-mp-version.in ${PROJECT_BINARY_DIR}/x86_64-${GXXNAME})

if( XCODE_PLUGIN_PATH )
    CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/LLVM_GCC.xcspec.in ${PROJECT_BINARY_DIR}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcspec)
    CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/GCC.xcspec.in ${PROJECT_BINARY_DIR}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcspec)
    CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/Info.plist.llvmgcc.in ${PROJECT_BINARY_DIR}/Info.plist.llvmgcc)
    CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/Info.plist.gcc.in ${PROJECT_BINARY_DIR}/Info.plist.gcc)
    INSTALL(FILES
            ${PROJECT_BINARY_DIR}/Info.plist.llvmgcc
            DESTINATION ${XCODE_PLUGIN_PATH}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE
            RENAME Info.plist )
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/version.plist 
            DESTINATION ${XCODE_PLUGIN_PATH}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES 
            ${PROJECT_BINARY_DIR}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcspec
            DESTINATION ${XCODE_PLUGIN_PATH}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/LLVM\ GCC.strings
            DESTINATION ${XCODE_PLUGIN_PATH}/LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources/English.lproj
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE
            RENAME LLVM\ GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.strings)
    INSTALL(FILES
            ${PROJECT_BINARY_DIR}/Info.plist.gcc
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE
            RENAME Info.plist )
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/version.plist 
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES 
            ${PROJECT_BINARY_DIR}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcspec
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/GCC\ Generic.xcspec
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/InfoPlist.strings
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources/English.lproj
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/GCC\ Generic.strings
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources/English.lproj
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE)
    INSTALL(FILES
            ${PROJECT_SOURCE_DIR}/GCC.strings
            DESTINATION ${XCODE_PLUGIN_PATH}/GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.xcplugin/Contents/Resources/English.lproj
            PERMISSIONS WORLD_READ GROUP_READ OWNER_READ OWNER_WRITE
            RENAME GCC\ ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}.strings)
endif( XCODE_PLUGIN_PATH )

#INSTALL(FILES ${PROJECT_BINARY_DIR}/i386-${GCCNAME}
#              ${PROJECT_BINARY_DIR}/i386-${GXXNAME}
#              ${PROJECT_BINARY_DIR}/x86_64-${GCCNAME}
#              ${PROJECT_BINARY_DIR}/x86_64-${GXXNAME}
#        DESTINATION bin 
#        PERMISSIONS WORLD_READ WORLD_EXECUTE OWNER_WRITE OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE)

# this will overwrite the gcc-mp-* executable
INSTALL(TARGETS ${GCCNAME} ${GXXNAME} DESTINATION bin)
