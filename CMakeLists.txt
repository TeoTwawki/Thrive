# Based on a file from the Ogre Wiki Tutorial Framework
#      http://www.ogre3d.org/tikiwiki/
#
# Modified as part of the Thrive project
# As of the Leviathan engine update this has been mostly rewritten
#-----------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.0)

project(Thrive)



###################
# Cache Variables #
###################

set(ASSET_DIRECTORY "${CMAKE_SOURCE_DIR}/assets"
  CACHE PATH "Path to assets"
  )

# This isn't really meant to be modified
set(LEVIATHAN_SRC "${PROJECT_SOURCE_DIR}/ThirdParty/Leviathan" CACHE FILEPATH
  "Path to leviathan source folder")

if(NOT IS_DIRECTORY "${ASSET_DIRECTORY}/models")
    message(FATAL_ERROR 
"Could not find assets in ${ASSET_DIRECTORY}.  
Please run the SetupThrive.rb script to download them.
Use 'thrive' for the username and password
"
    )
endif()


###########
# Options #
###########

option(USE_BREAKPAD "For enabling breakpad crash handling, set off for local debugging" OFF)
option(COPY_BOOST_TO_PACKAGE "If on copies all boost libraries to package" ON)


###########################
# Leviathan Project Setup #
###########################

# Set up cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMake"
  "${LEVIATHAN_SRC}/CMake"
  "${LEVIATHAN_SRC}/LeviathanCMake")

set(LEVIATHAN_SKIP_OPTIONAL_ASSETS ON)

include(LeviathanCompileOptions)
include(LeviathanSetRPath)

set(LEVIATHAN_FULL_BUILD ON)

set(USE_ANGELSCRIPT ON)
set(USE_BOOST ON)
set(USE_OGRE ON)
set(USE_NEWTON ON)
set(USE_SFML ON)
set(USE_SDL2 ON)

include(LeviathanFindLibraries)

include(LeviathanUtility)

# Leviathan Required settings
DefinePreprocessorMacro(USE_BREAKPAD LEVIATHAN_USING_BREAKPAD)

include(LeviathanDefaultFileCopy)
include(LeviathanSetRPath)

# -Wl,-rpath-link is used to suppress warnings from games linking gainst libEngine.so
if(UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wl,-rpath-link,${CMAKE_BINARY_DIR}/bin/lib")
endif()

# Links to libs

# Copy engine lib
if(WIN32)
  file(COPY "${LEVIATHAN_SRC}/build/bin/Engine.dll" DESTINATION "${CMAKE_BINARY_DIR}/bin")
  file(COPY "${LEVIATHAN_SRC}/build/bin/Engine.pdb" DESTINATION "${CMAKE_BINARY_DIR}/bin")
  install(FILES "${LEVIATHAN_SRC}/build/bin/Engine.dll" DESTINATION "bin")
else()
  file(COPY "${LEVIATHAN_SRC}/build/bin/libEngine.so" DESTINATION "${CMAKE_BINARY_DIR}/bin")
  install(FILES "${LEVIATHAN_SRC}/build/bin/libEngine.so" DESTINATION "bin")
endif()

# Link own bin directory
link_directories("${CMAKE_BINARY_DIR}/bin")

# And engine
link_directories("${LEVIATHAN_SRC}/build/bin")

# Version
set(PROGRAM_VERSION_STABLE 0)
set(PROGRAM_VERSION_MAJOR 4)
set(PROGRAM_VERSION_MINOR 0)
set(PROGRAM_VERSION_PATCH 0)
set(PROGRAM_VERSION_SUFFIX "")

set(PROGRAM_VERSION_STR ${PROGRAM_VERSION_STABLE}.${PROGRAM_VERSION_MAJOR}.${PROGRAM_VERSION_MINOR}.${PROGRAM_VERSION_PATCH}${PROGRAM_VERSION_SUFFIX})
set(PROGRAM_VERSION ${PROGRAM_VERSION_STABLE}.${PROGRAM_VERSION_MAJOR}${PROGRAM_VERSION_MINOR}${PROGRAM_VERSION_PATCH})

set(PROGRAM_VERSIONS "\"${PROGRAM_VERSION_STR}\"")
set(PROGRAM_VERSIONS_ANSI "\"${PROGRAM_VERSION_STR}\"")

set(ENGINECONFIGURATION				"./EngineConf.conf")
set(GAMENAMEIDENTIFICATION			Thrive)
set(GAMEVERSIONIDENTIFICATION		GAME_VERSIONS)
set(ProgramNamespace thrive)
# Currently there is no master server
# set(PROGRAMMASTERSERVERINFO			"MasterServerInformation(\"ThriveMasters.txt\", \"Thrive_\" GAME_VERSIONS, \"http://revolutionarygamesstudio.com/\", \"/Thrive/MastersList.png\", \"ThriveAccountCrecentials.txt\", false)")
set(PROGRAMMASTERSERVERINFO			"MasterServerInformation()")

# Include dirs
include_directories("${LEVIATHAN_SRC}/Engine")

set(DEPENDENT_LIBS ${LEVIATHAN_APPLICATION_LIBRARIES})


# Thrive compile setup
if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-unused-function -Wno-error=redundant-decls")
endif()
include_directories(
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

add_subdirectory(src)

add_subdirectory(test)

# Set the main executable as the startup project
set_property(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" PROPERTY
    VS_STARTUP_PROJECT Thrive)

# Copy script files to the bin folder
file(GLOB THRIVE_SCRIPTS "scripts/*")
file(COPY ${THRIVE_SCRIPTS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Scripts")
InstallContentsOfFolder("scripts" "bin/Data/Scripts")

# Right now we only use what the engine needs
# # Copy javascript libraries
# file(GLOB THRIVE_VENDOR_SCRIPTS "assets/jsvendor/*")
# file(COPY ${THRIVE_VENDOR_SCRIPTS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/jsvendor")

# Copy other data assets
file(GLOB THRIVE_TEXTURES "assets/textures/*")
file(COPY ${THRIVE_TEXTURES} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Textures/")
InstallContentsOfFolder("assets/textures" "bin/Data/Textures")


file(GLOB THRIVE_FONTS "assets/fonts/*")
file(COPY ${THRIVE_FONTS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Fonts/")
InstallContentsOfFolder("assets/fonts" "bin/Data/Fonts")


file(GLOB THRIVE_MATERIALS "assets/materials/*")
file(COPY ${THRIVE_MATERIALS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Materials/")
InstallContentsOfFolder("assets/materials" "bin/Data/Materials")

file(GLOB THRIVE_MODELS "assets/models/*")
file(COPY ${THRIVE_MODELS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Models/")
InstallContentsOfFolder("assets/models" "bin/Data/Models")

file(GLOB THRIVE_SOUNDS "assets/sounds/*")
file(COPY ${THRIVE_SOUNDS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Sound/")
InstallContentsOfFolder("assets/sounds" "bin/Data/Sound")

file(GLOB THRIVE_VIDEOS "assets/videos/*")
file(COPY ${THRIVE_VIDEOS} DESTINATION "${PROJECT_BINARY_DIR}/bin/Data/Videos/")
InstallContentsOfFolder("assets/videos" "bin/Data/Videos")

file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/bin/Data/Screenshots")
file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/bin/Data/Cache")


# #######
# # Lua #
# #######

# option(LUA_CHECKS "when ON enables additional Lua safety
#        checks. Lowers performance so only use while debugging"
#        OFF)

# if(LUA_CHECKS)

#   add_definitions(-DSOL_CHECK_ARGUMENTS)
  
# endif()

# include_directories(
#   "${CMAKE_CURRENT_SOURCE_DIR}/contrib/lua"
#   )

# link_directories(
#   "${CMAKE_CURRENT_SOURCE_DIR}/contrib/lua/luajit/src"
#   )

# set(LUA_FOUND TRUE)

# if(WIN32)
#   set(LUA_LIBRARIES lua51.dll)
# else()
#   set(LUA_LIBRARIES luajit.a dl)
# endif()





#################
# Documentation #
#################

set(DOC_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/doc" CACHE PATH "Documentation output directory.")
set(DOC_INPUT "${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/doc")

set( DOXYGEN_CONFIG_FILE
    "${CMAKE_CURRENT_SOURCE_DIR}/doc/doxygen.cfg.in"
)

find_package(Doxygen)

if(DOXYGEN_FOUND)
    configure_file("${DOXYGEN_CONFIG_FILE}" "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg" @ONLY)
    add_custom_target(doc
        "${DOXYGEN_EXECUTABLE}" "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg"
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/doc"
        COMMENT "Generating API documentation with Doxygen" VERBATIM
        SOURCES "${DOXYGEN_CONFIG_FILE}"
    )
endif()
