#: # CMakeLists.txt ([source](../appskeleton/CMakeLists.txt))
#: 
#: `CMakeLists.txt` contains the build configuration that can be used to build
#: your App without any IDE or to generate project files for your specific IDE.
#: 
#: ## Required settings for all apps
cmake_minimum_required(VERSION 3.5)

#: This sets the minimum required CMake version.
#: Here, [CMake 3.5](https://cmake.org/cmake/help/latest/release/3.5.html#modules)
#: was chosen because it is the first version to support imported targets
#: (e.g. `boost::thread`). Before setting this to a higher version, please check
#: if it has arrived in [debian stable](https://packages.debian.org/stable/cmake).
project(SMIEyetracker
	LANGUAGES CXX
	VERSION 0.1)

#: [project](https://cmake.org/cmake/help/latest/command/project.html) sets the
#: name of the app, the languages used and the version. The version is later on
#: used in the packages CMake creates for your app.
#: 
#: ## Finding liblsl
#: 
#: Your app most likely requires liblsl, so CMake has to find it unless it already
#: knows about liblsl (because your app is included in a build for the whole tree)


# Maybe fix automoc fail
set(CMAKE_INCLUDE_CURRENT_DIR ON)

#: We skip searching for liblsl if CMake already knows about it:
# set up LSL if not done already
if(NOT TARGET LSL::lsl)

#: Users can define `LSL_INSTALL_ROOT` to tell CMake where to look for
#: `LSLConfig.cmake`. Normally we use paths with forward slashes (`/`), which are
#: also accepted by Windows unless the path also contains backslashes (`\\`).
#: Therefore we convert the user supplied path to use forward slashes
#: (`TO_CMAKE_PATH`).
	# when building out of tree LSL_ROOT needs to be specified on the cmd line
	file(TO_CMAKE_PATH "${LSL_INSTALL_ROOT}" LSL_INSTALL_ROOT)

#: We also look in `../../LSL/liblsl/build/install`, since that's where liblsl
#: gets installed by default.
	list(APPEND LSL_INSTALL_ROOT "${CMAKE_CURRENT_LIST_DIR}/../../LSL/liblsl/build/install")

#: 
#: We search for the package config in the two possible subfolders.
#: Normally, we add `REQUIRED` (=error if package isn't found) to `find_package`,
#: not `QUIET` (=not even a warning if it isn't found), but in this case we want
#: to print a more helpful error message:
	find_package(LSL HINTS ${LSL_INSTALL_ROOT}/share/LSL/ ${LSL_INSTALL_ROOT}/LSL/share/LSL QUIET)
	if(NOT LSL_FOUND)
		message(FATAL_ERROR "Precompiled LSL was not found. See https://github.com/labstreaminglayer/labstreaminglayer/blob/master/doc/BUILD.md#lsl_install_root for more information.")
	endif()

#: `LSLCMake.cmake` contains helper functions to find Qt and Boost and create
#: redistributable archives for each app. We add the liblsl folder to the module
#: search path and include it

	list(APPEND CMAKE_MODULE_PATH ${LSL_DIR})
	message(STATUS "Looking for LSLCMake in ${LSL_DIR}")
	include(LSLCMake)
endif()

# include boost path (headers and libraries):
include_directories("C:/local/boost_1_69_0")
if (CMAKE_CL_64)
  link_directories("C:/local/boost_1_69_0/lib64-msvc-14.1")
endif()

if(NOT CMAKE_CL_64)
  link_directories("C:/local/boost_1_69_0/lib32-msvc-14.1")
endif()
  
  # include allegro path (TODO CHECK):
include_directories("C:/local/allegro/include")
link_directories("C:/local/allegro/lib")



#: ## Finding vendor SDKs
#: 
#: More often than not, vendors offer a library files (`.so`, `.dylib`, `.dll`)
#: and a header file (`.h`). CMake can search for these files and create an
#: imported target, so all compilers get the correct settings where to look for
#: them and how to link them.

# create an imported target we later add the required information to
#add_library(vendorsdk SHARED IMPORTED)

# create an imported target we later add the required information to
add_library(vendorsdk STATIC IMPORTED)
# find the vendorsdk_research.h header in the subfolder vendorsdk/include
find_path(vendorsdk_SDK_INCLUDE_PATH
	name "iViewXAPI.h"
	PATHS "./include"
	NO_DEFAULT_PATH   # otherwise found in C:/Programme[..], doesn't work from there
)
if(NOT vendorsdk_SDK_INCLUDE_PATH)
	message(FATAL_ERROR "vendorsdk SDK INCLUDE PATH not found")
endif()

# find a shared library called e.g. vendorsdk_research.lib or libvendorsdk_research.so
# in vendorsdk/lib
if (CMAKE_CL_64)
  find_library(vendorsdk_SDK_LIB iViewXAPI64
  	PATHS "./include"
  	NO_DEFAULT_PATH   # otherwise found in C:/Programme[..], doesn't work from there
  )
  message("-- Using 64-Bit libraries")
endif()

if (NOT CMAKE_CL_64)
  find_library(vendorsdk_SDK_LIB iViewXAPI
  	PATHS "./include"
  	NO_DEFAULT_PATH   # otherwise found in C:/Programme[..], doesn't work from there
  )
  message("-- Using 32-Bit libraries")
endif()

# Windows needs additional help
get_filename_component(libext vendorsdk_SDK_LIB EXT)
if(libext STREQUAL ".lib")
	set_target_properties(vendorsdk PROPERTIES IMPORTED_IMPLIB ${vendorsdk_SDK_LIB})
	string(REPLACE ".lib" ".dll" vendorsdk_SDK_LIB ${vendorsdk_SDK_LIB})
endif()
 #add the paths we just found to the vendorsdk library target
set_target_properties(vendorsdk PROPERTIES
	INTERFACE_INCLUDE_DIRECTORIES ${vendorsdk_SDK_INCLUDE_PATH}
	IMPORTED_LOCATION ${vendorsdk_SDK_LIB}
)
if(NOT vendorsdk_SDK_LIB)
	message(FATAL_ERROR "vendorsdk SDK library not found")
endif()
#: ## Using Qt
#:
#: To import Qt, just call `find_package` with the components your app uses
#: (usually just `Widgets`).

find_package(Qt5 REQUIRED COMPONENTS Widgets Core)
find_package(Boost REQUIRED)

#: ## Native threads
#:
#: Native `std::thread`s still require a platform thread library. CMake
#: can find and link to it with the `Threads` package (link your executable with
#: `Threads::Threads` afterwards).

find_package(Threads REQUIRED)

#: ## Creating executables for your app
#: 
#: Your app can have multiple executables. All of them are set up the same way:

add_executable(${PROJECT_NAME} MACOSX_BUNDLE WIN32
	main.cpp
	mainwindow.cpp
	mainwindow.h
	mainwindow.ui
)

#file(GLOB ALLEGRO_LIBS "C:/local/allegro/lib/*.lib")

#: If everything succeeds, you can link your app with the vendor SDK
#: by linking to the imported target.
#: 

target_link_libraries(${PROJECT_NAME}
	#PRIVATE
	Qt5::Widgets
	Threads::Threads
	LSL::lsl
	vendorsdk
	#${ALLEGRO_LIBS}
)




set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
# target_compile_features(${PROJECT_NAME} PRIVATE cxx_auto_type cxx_lambda_init_captures)

#: [add_executable](https://cmake.org/cmake/help/latest/command/add_executable.html)
#: tells CMake to take all files listed (even Qt `.ui` files), compile them and
#: link them together. `MACOSX_BUNDLE` creates a bundle on OS X, `WIN32` tells
#: Windows compilers not to show a command line window when launching the app.
#: 
#: Using `${PROJECT_NAME}` as a placeholder for the executable name  makes it easier
#: to reuse parts of the `CMakeLists.txt` in other projects.
#: 
#: [target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)
#: tells CMake to add the include paths (and if necessary `#define`s) to the
#: compiler command line and link to the libraries when producing a binary.
#:
#: If you want to use newer C++ features, either set the target standard version via
#: [`set_property(... CXX_STANDARD 11)`](https://cmake.org/cmake/help/latest/prop_tgt/CXX_STANDARD.html)
#: or explicitly enable features you need with 
#: [`target_compile_features`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)
#: (see [`CMAKE_CXX_KNOWN_FEATURES`](https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html#prop_gbl:CMAKE_CXX_KNOWN_FEATURES)
#: for a list.
#: 
#: ## Setting up deployment
#: 
#: You can also let CMake generate a zip / dmg file with everything needed to run
#: your app:

installLSLApp(${PROJECT_NAME})

#installLSLAuxFiles(${PROJECT_NAME}
#	${PROJECT_NAME}.cfg
#)

LSLGenerateCPackConfig()
#: `installLSLApp` creates an install target for the binary target, so that the
#: CMake install target creates a directory for your app and copies the binary
#: and all needed libraries (including Qt) to this folder.
#: 
#: `installLSLAuxFiles` copies additional files needed (e.g. config files) to the
#: distribution directory.
#: 
#: `LSLGenerateCPackConfig` (has to be the last line!) generates a
#: [CPack](https://cmake.org/Wiki/CMake:Packaging_With_CPack) configuration.
#: CPack will then create packages (`.deb` on Linux, `.dmg` on OS X, `.zip` on
#: Windows) that are easy to send someone and install on another computer.