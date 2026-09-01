cmake_minimum_required(VERSION "3.19.2")

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

######################################## set properties  ########################################

# set ON if you want a prod build
set(PRODUCTION_BUILD OFF CACHE BOOL "Make this a production build" FORCE)

# optimizations only for prod
if(PRODUCTION_BUILD)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
else()
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION FALSE)
endif()

if(MSVC)
    add_compile_options(/arch:AVX2)
endif()

# static stdlib linking
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Release>:Release>")

# static sfml linking
set(SFML_STATIC_LIBRARIES TRUE)
add_compile_definitions(SFML_STATIC)

# makes glfw statically linked and disables unwanted features
# set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
# set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
# set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
# set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
# set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

# show compile commands in cmake output
set(CMAKE_EXPORT_COMPILE_COMMANDS)

# sets visual studio startup project
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${CMAKE_PROJECT_NAME})

######################################## fetching sources of third patry libraries ########################################

include(FetchContent)

FetchContent_Declare(
	sfml
	GIT_REPOSITORY https://github.com/SFML/SFML
	GIT_TAG 3.1.0
	SOURCE_DIR     ${CMAKE_CURRENT_SOURCE_DIR}/libs/sfml
	OVERRIDE_FIND_PACKAGE
)

FetchContent_Declare(
	imgui
	GIT_REPOSITORY https://github.com/ocornut/imgui
	GIT_TAG v1.91.9
	SOURCE_DIR     ${CMAKE_CURRENT_SOURCE_DIR}/libs/imgui
	OVERRIDE_FIND_PACKAGE
)

FetchContent_Declare(
	imgui-sfml
	GIT_REPOSITORY https://github.com/SFML/imgui-sfml
	GIT_TAG v3.0
	SOURCE_DIR     ${CMAKE_CURRENT_SOURCE_DIR}/libs/imgui-sfml
	OVERRIDE_FIND_PACKAGE
)

FetchContent_MakeAvailable(sfml)

FetchContent_GetProperties(imgui)
if(NOT imgui_POPULATED)
    FetchContent_Populate(imgui)
endif()
set(IMGUI_DIR "${imgui_SOURCE_DIR}" CACHE PATH "Path to imgui" FORCE)

FetchContent_MakeAvailable(imgui-sfml)

######################################## game files ########################################

project(GameTemplate)

# add sources to the project
file(GLOB_RECURSE GAME_SOURCES CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(${CMAKE_PROJECT_NAME} ${GAME_SOURCES})

# do not use automatic opengl headers include
target_compile_definitions(${CMAKE_PROJECT_NAME} PUBLIC GLFW_INCLUDE_NONE=1)

######################################## compilation settings ########################################

if(PRODUCTION_BUILD)
	# setup the RESOURCES_PATH macro to be in the root folder of your exe
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC RESOURCES_PATH="./resources/")

	# remove the option to debug asserts.
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC PRODUCTION_BUILD=1)
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC DEVELOPLEMT_BUILD=0)
else()
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC RESOURCES_PATH="${CMAKE_CURRENT_SOURCE_DIR}/resources/")
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC PRODUCTION_BUILD=0)
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC DEVELOPLEMT_BUILD=1)
endif()

if(MSVC)
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC _CRT_SECURE_NO_WARNINGS)
	set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup") # no cmd window
endif()

if (WIN32)
	target_compile_options("${CMAKE_PROJECT_NAME}" PRIVATE /UUNICODE /U_UNICODE) # force remove unicode
endif()

######################################## including and linking libraries ########################################
 
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/libs/sfml/include)
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/libs/imgui)
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/libs/imgui-SFML)

target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC SFML::Graphics SFML::Audio SFML::System SFML::Window ImGui-SFML::ImGui-SFML)
