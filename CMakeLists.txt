cmake_minimum_required(VERSION 3.14.0)
project(StreamFormat VERSION 0.1 DESCRIPTION "")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_INSTALL_PREFIX}/lib/${CMAKE_SYSTEM_PROCESSOR}/${CMAKE_BUILD_TYPE})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_INSTALL_PREFIX}/lib/${CMAKE_SYSTEM_PROCESSOR}/${CMAKE_BUILD_TYPE})


set(DEPENDENCIES Stream)
set(INC_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(SRC_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(INC_LIST
		${INC_ROOT}/${PROJECT_NAME}/Base.h
		${INC_ROOT}/${PROJECT_NAME}/Dot.h
		${INC_ROOT}/${PROJECT_NAME}/String.h
		)
set(SRC_LIST
		${SRC_ROOT}/Base.cpp
		${SRC_ROOT}/Dot.cpp
		${SRC_ROOT}/String.cpp
		${SRC_ROOT}/String.hpp
		)


add_library(${PROJECT_NAME}OBJ OBJECT)
add_library(${PROJECT_NAME} SHARED)
target_include_directories(${PROJECT_NAME}OBJ PUBLIC include)
target_include_directories(${PROJECT_NAME} PUBLIC include)
target_sources(${PROJECT_NAME}OBJ PUBLIC ${INC_LIST} PRIVATE ${SRC_LIST})
target_sources(${PROJECT_NAME} PUBLIC ${INC_LIST} PRIVATE ${SRC_LIST})

foreach(T IN LISTS DEPENDENCIES)
	if (NOT TARGET ${T})
		add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../${T} ${CMAKE_CURRENT_BINARY_DIR}/../../../${T}/${CMAKE_SYSTEM_PROCESSOR}/${CMAKE_BUILD_TYPE})
	endif ()
endforeach()
target_link_libraries(${PROJECT_NAME} PUBLIC ${DEPENDENCIES})
list(TRANSFORM DEPENDENCIES APPEND OBJ)
target_link_libraries(${PROJECT_NAME}OBJ PUBLIC ${DEPENDENCIES})


if (CMAKE_BUILD_TYPE MATCHES Debug)
	enable_testing()
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/test ${CMAKE_CURRENT_BINARY_DIR}/Testing)
endif()


if ((DEFINED DOC_ROOT) AND (NOT ${DOC_ROOT} STREQUAL ""))
	find_package(Doxygen)
	if (DOXYGEN_FOUND)
		set(DOXYGEN_IN_CFG ${CMAKE_CURRENT_SOURCE_DIR}/doxygen.in.cfg)
		set(DOXYGEN_OUTPUT_DIR ${DOC_ROOT}/${PROJECT_NAME})
		set(DOXYGEN_CFG ${DOXYGEN_OUTPUT_DIR}/doxygen.cfg)

		configure_file(${DOXYGEN_IN_CFG} ${DOXYGEN_CFG} @ONLY)
		message("Doxygen configuration ${DOXYGEN_CFG} generated.")

		add_custom_target(${PROJECT_NAME}Doc
				COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_CFG}
				WORKING_DIRECTORY ${DOXYGEN_OUTPUT_DIR}
				COMMENT "Generating ${PROJECT_NAME} documentation with Doxygen"
				VERBATIM)
	else (DOXYGEN_FOUND)
		message("Doxygen need to be installed to generate the doxygen documentation")
	endif (DOXYGEN_FOUND)
endif()