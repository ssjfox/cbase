# Copyright 2021. (주)웨어그램 all rights reserved.
# 빌드 CMakeLists.txt

cmake_minimum_required(VERSION 3.0.0)

project($ENV{OUTPUT} VERSION 0.1.0)

SET(CMAKE_BUILD_TYPE "")   # "Release" "Debug"
SET(CMAKE_CXX_FLAGS_DEBUG "")
SET(CMAKE_C_FLAGS_DEBUG "")
SET(CMAKE_CXX_FLAGS "")
SET(CMAKE_C_FLAGS "")

MESSAGE("out $ENV{OUTPUT}")
MESSAGE("make : ${CMAKE_MAKE_PROGRAM} $ENV{COMPILER} $ENV{GCC} $ENV{GXX}")

# MESSAGE("CMAKE_BUILD_TYPE : ${CMAKE_BUILD_TYPE}")
# SET(CMAKE_BUILD_TYPE "Release") "Debug" # 사전 준비된 컴파일 옵션이 적용된다.

if($ENV{COMPILER} STREQUAL "gcc")
    SET(CMAKE_C_COMPILER "$ENV{GCC}")
    SET(CMAKE_CXX_COMPILER "$ENV{GXX}")
elseif($ENV{COMPILER} STREQUAL "msvc")
	SET(CMAKE_C_COMPILER "cl.exe")
	SET(CMAKE_CXX_COMPILER "cl.exe")
	SET(MSVC_TOOLSET_VERSION $ENV{MSVC_TOOLSET_VERSION})
endif()

MESSAGE( "${CMAKE_BUILD_TYPE}")

file(STRINGS "${CMAKE_SOURCE_DIR}/../source_list" PROJECT_SOURCE_LIST )
list(FILTER PROJECT_SOURCE_LIST EXCLUDE REGEX "^#")
foreach( PROJECT_SOURCE_ITEM ${PROJECT_SOURCE_LIST}  )
    SET(PROJECT_SOURCE_ITEM "${CMAKE_SOURCE_DIR}/../${PROJECT_SOURCE_ITEM}")
    file(GLOB_RECURSE PROJECT_SOURCE_ITEM_FILES ${PROJECT_SOURCE_ITEM})
    foreach( PROJECT_SOURCE_ITEM_FILE ${PROJECT_SOURCE_ITEM_FILES}  )
        get_filename_component( PROJECT_SOURCE_ITEM_FILEDIR ${PROJECT_SOURCE_ITEM_FILE} DIRECTORY )
        get_filename_component( PROJECT_SOURCE_ITEM_FILENAME ${PROJECT_SOURCE_ITEM_FILE} NAME_WE )
        get_filename_component( PROJECT_SOURCE_ITEM_FILEEXT ${PROJECT_SOURCE_ITEM_FILE} EXT )
        if( "${PROJECT_SOURCE_ITEM_FILEEXT}" STREQUAL ".ui")
            if( "$ENV{B_CLEAN}" STREQUAL "clean" )
                execute_process(COMMAND mkdir -p ${PROJECT_SOURCE_ITEM_FILEDIR}/QT)
                execute_process(COMMAND uic ${PROJECT_SOURCE_ITEM_FILE} -o ${PROJECT_SOURCE_ITEM_FILEDIR}/QT/${PROJECT_SOURCE_ITEM_FILENAME}.hxx)
            endif()
        elseif( "${PROJECT_SOURCE_ITEM_FILEEXT}" STREQUAL ".hpp")
            if( "$ENV{B_CLEAN}" STREQUAL "clean" )
                execute_process(COMMAND mkdir -p ${PROJECT_SOURCE_ITEM_FILEDIR}/QT)
                execute_process(COMMAND moc ${PROJECT_SOURCE_ITEM_FILE} -o ${PROJECT_SOURCE_ITEM_FILEDIR}/QT/${PROJECT_SOURCE_ITEM_FILENAME}.cxx)
                list (APPEND PROJECT_SOURCE_FILES ${PROJECT_SOURCE_ITEM_FILEDIR}/QT/${PROJECT_SOURCE_ITEM_FILENAME}.cxx)
            endif()
        else()
            list (APPEND PROJECT_SOURCE_FILES ${PROJECT_SOURCE_ITEM_FILE})
        endif()
    endforeach()
endforeach()

macro( LOAD_CONFIG CONFIG_DIR)
    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/compile_option")
        message(STATUS "COMPILE OPTION FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/compile_option")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/compile_option" COMPILE_OPTIONS )
        list(FILTER COMPILE_OPTIONS EXCLUDE REGEX "^#")
        foreach( COMPILE_OPTION ${COMPILE_OPTIONS}  )
            if( NOT ${COMPILE_OPTION} STREQUAL "" AND NOT ${COMPILE_OPTION} STREQUAL " ")
                add_compile_options("${COMPILE_OPTION}")
                message(STATUS "COMPILE_OPTION : ${COMPILE_OPTION}")
            endif()
        endforeach()
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/link_option")
        message(STATUS "LINK OPTION FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/link_option")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/link_option" LINK_OPTIONS )
        list(FILTER LINK_OPTIONS EXCLUDE REGEX "^#")
        foreach( LINK_OPTION ${LINK_OPTIONS}  )
            if( NOT ${LINK_OPTION} STREQUAL "" AND NOT ${LINK_OPTION} STREQUAL " ")
                add_link_options("${LINK_OPTION}")
                message(STATUS "LINK_OPTION : ${LINK_OPTION}")
            endif()
        endforeach()
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_includes")
        message(STATUS "INCLUDE DIRECTORY FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_includes")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_includes" INCLUDE_DIRECTORIES )
        list(FILTER INCLUDE_DIRECTORIES EXCLUDE REGEX "^#")
        foreach( INCLUDE_DIRECTORY ${INCLUDE_DIRECTORIES}  )
            string(SUBSTRING ${INCLUDE_DIRECTORY} 0 1 LRDIR)
            string(SUBSTRING ${INCLUDE_DIRECTORY} 1 1 WRDIR)
            if( ${LRDIR} STREQUAL "/" OR ${LRDIR} STREQUAL "~" OR ${WRDIR} STREQUAL ":")
                list(APPEND PROJECT_INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORY})
            else()
                list(APPEND PROJECT_INCLUDE_DIRECTORIES ../${INCLUDE_DIRECTORY})
            endif()
        endforeach()
        include_directories( ${PROJECT_INCLUDE_DIRECTORIES} )
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_libraries")
        message(STATUS "LINK DIRECTORY FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_libraries")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/directories_for_libraries" LINK_DIRECTORIES )
        list(FILTER LINK_DIRECTORIES EXCLUDE REGEX "^#")
        foreach( LINK_DIRECTORY ${LINK_DIRECTORIES}  )
            string(SUBSTRING ${LINK_DIRECTORY} 0 1 LRDIR)
            string(SUBSTRING ${LINK_DIRECTORY} 1 1 WRDIR)
            if( ${LRDIR} STREQUAL "/" OR ${LRDIR} STREQUAL "~" OR ${WRDIR} STREQUAL ":")
                list(APPEND PROJECT_LINK_DIRECTORIES ${LINK_DIRECTORY})
            else()
                list(APPEND PROJECT_LINK_DIRECTORIES ../${LINK_DIRECTORY})
            endif()
        endforeach()
        link_directories( ${PROJECT_LINK_DIRECTORIES} )
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/libraries")
        message(STATUS "LINK LIBRARY FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/libraries")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/libraries" PROJECT_LINK_LIBRARIES )
        list(FILTER PROJECT_LINK_LIBRARIES EXCLUDE REGEX "^#")
        message(STATUS "LINK LIBRARY FILE : ${PROJECT_LINK_LIBRARIES}")
        link_libraries( ${PROJECT_LINK_LIBRARIES} )
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/macros")
        message(STATUS "MACRO FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/macros")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/macros" PROJECT_DEFINITIONS )
        list(FILTER PROJECT_DEFINITIONS EXCLUDE REGEX "^#")
        foreach( DEFINITION ${PROJECT_DEFINITIONS}  )
            if( NOT ${DEFINITION} STREQUAL "" AND NOT ${DEFINITION} STREQUAL " ")
                add_definitions( -D${DEFINITION} )
                message(STATUS "-- MACRO ${DEFINITION}")
            endif()
        endforeach()
    endif()

    if(EXISTS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/ignore_source_filter")
        message(STATUS "IGNORE SOURCE FILTER FILE : ${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/ignore_source_filter")
        file(STRINGS "${CMAKE_SOURCE_DIR}/../config/${CONFIG_DIR}/ignore_source_filter" IGNORE_SOURCE_FILTER_LIST )
        list(FILTER IGNORE_SOURCE_FILTER_LIST EXCLUDE REGEX "^#")
        foreach( IGNORE_SOURCE_FILTER ${IGNORE_SOURCE_FILTER_LIST}  )
            list(FILTER PROJECT_SOURCE_FILES EXCLUDE REGEX ${IGNORE_SOURCE_FILTER})
            message(STATUS "-- IGNORE_SOURCE_FILTER ${IGNORE_SOURCE_FILTER}")
        endforeach()
    endif()

endmacro()

load_config("Common")
load_config("Common_$ENV{BUILD_TYPE}")
load_config("$ENV{SYSTEM_NAME}_common")
load_config("$ENV{SYSTEM_NAME}_common_$ENV{BUILD_TYPE}")
load_config("$ENV{SYSTEM_NAME}_$ENV{HOSTTYPE}")
load_config("$ENV{SYSTEM_NAME}_$ENV{HOSTTYPE}_$ENV{BUILD_TYPE}")

# foreach( PROJECT_SOURCE_FILE ${PROJECT_SOURCE_FILES}  )
#     MESSAGE( "PROJECT_SOURCE_FILE: " ${PROJECT_SOURCE_FILE} )
# endforeach()

SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/../bin/$ENV{ENV_DIR}")
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/../bin/$ENV{ENV_DIR}")
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/../bin/$ENV{ENV_DIR}")

if($ENV{B_TYPE} STREQUAL "excutable binary")
    # message( ${PROJECT_SOURCE_FILES} )
	add_executable(${PROJECT_NAME} ${PROJECT_SOURCE_FILES})
else()
	if($ENV{B_LIBRARY_TEST} STREQUAL "library test")
		add_executable(test_${PROJECT_NAME} ${PROJECT_SOURCE_FILES})
	else()
		if($ENV{B_TYPE} STREQUAL "static library")
			add_library(${PROJECT_NAME} ${PROJECT_SOURCE_FILES})
		elseif($ENV{B_TYPE} STREQUAL "shared library")
			add_library(${PROJECT_NAME} SHARED ${PROJECT_SOURCE_FILES})
		endif()
	endif()
endif()

# add_custom_target ( run
# 	COMMAND ${PROJECT_NAME}
# 	USES_TERMINAL
# )
