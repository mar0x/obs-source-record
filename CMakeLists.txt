# --- Detect if the plugin is build out of tree or not ---
if(CMAKE_PROJECT_NAME STREQUAL "obs-studio")
  set(BUILD_OUT_OF_TREE OFF)
else()
  set(BUILD_OUT_OF_TREE ON)
  cmake_minimum_required(VERSION 3.18)
endif()

include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/common/bootstrap.cmake" NO_POLICY_SCOPE)

project(obs-source-record VERSION 0.4.8)
set(PROJECT_FULL_NAME "Source Record")

# Set new UUIDs when you start to create a new plugin.
set(MACOS_PACKAGE_UUID "3EC43C98-466F-4B74-8CB4-17B90EB0B29F")
set(MACOS_INSTALLER_UUID "599A0945-8DDB-4E81-8A4D-C399D04BCCC5")

add_library(${PROJECT_NAME} MODULE)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/version.h.in ${CMAKE_CURRENT_SOURCE_DIR}/version.h)

target_sources(${PROJECT_NAME} PRIVATE
	source-record.c
	source-record.h)


if(BUILD_OUT_OF_TREE)
	include(compilerconfig)
	include(defaults)
	include(helpers)

	find_package(libobs REQUIRED)
	find_package(obs-frontend-api REQUIRED)

	set(OBS_FRONTEND_API_NAME "obs-frontend-api")
else()
	set(OBS_FRONTEND_API_NAME "frontend-api")
endif()
  
if(OS_WINDOWS)
	get_filename_component(ISS_FILES_DIR "${CMAKE_BINARY_DIR}\\..\\package" ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_FILES_DIR}" ISS_FILES_DIR)
	get_filename_component(ISS_PACKAGE_DIR "${CMAKE_PACKAGE_PREFIX}\\.." ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_PACKAGE_DIR}" ISS_PACKAGE_DIR)
	get_filename_component(ISS_SOURCE_DIR "${PROJECT_SOURCE_DIR}" ABSOLUTE)
	file(TO_NATIVE_PATH "${ISS_SOURCE_DIR}" ISS_SOURCE_DIR)
	configure_file("installer.iss.in"
			"${PROJECT_BINARY_DIR}/installer.iss"
		)
	configure_file(${CMAKE_CURRENT_SOURCE_DIR}/resource.rc.in ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.rc)
	target_sources(${PROJECT_NAME} PRIVATE 
		${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.rc)
	target_link_libraries(${PROJECT_NAME}
		w32-pthreads)
elseif(OS_MACOS)
		set(MACOS_BUNDLEID "com.exeldro.${PROJECT_NAME}")
		set(MACOSX_PLUGIN_GUI_IDENTIFIER "${MACOS_BUNDLEID}")
		set(MACOSX_PLUGIN_BUNDLE_VERSION "${PROJECT_VERSION}")
		set(MACOSX_PLUGIN_SHORT_VERSION_STRING "1")
		configure_file(cmake/bundle/macos/installer-macos.pkgproj.in ${CMAKE_BINARY_DIR}/installer-macos.generated.pkgproj)
		target_compile_options(${PROJECT_NAME} PRIVATE -Wall)
endif()

target_link_libraries(${PROJECT_NAME}
	OBS::${OBS_FRONTEND_API_NAME}
	OBS::libobs)

if(BUILD_OUT_OF_TREE)
    set_target_properties_plugin(${CMAKE_PROJECT_NAME} PROPERTIES OUTPUT_NAME ${_name})

else()
    target_include_directories(${PROJECT_NAME} PRIVATE
        "${CMAKE_SOURCE_DIR}/UI/obs-frontend-api")
	if(OBS_CMAKE_VERSION VERSION_GREATER_EQUAL 3.0.0)
		set_target_properties_obs(${PROJECT_NAME} PROPERTIES FOLDER "plugins/exeldro" PREFIX "")
	else()
		set_target_properties(${PROJECT_NAME} PROPERTIES FOLDER "plugins/exeldro")
	endif()
endif()
