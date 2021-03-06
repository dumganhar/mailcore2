set(CMAKE_INCLUDE_CURRENT_DIR ON)
IF(APPLE)
  set(dir "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk")
  IF(EXISTS "${dir}" AND IS_DIRECTORY "${dir}")
    set(SDKROOT "${dir}")
  ELSE()
    set(dir "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.8.sdk")
    IF(EXISTS "${dir}" AND IS_DIRECTORY "${dir}")
      set(SDKROOT "${dir}")
    ENDIF()
  ENDIF()
  
  IF(NOT SDKROOT)
    message(FATAL_ERROR "ERROR: Could not find Mac OSX SDK 10.8 or 10.9")
  ELSE()
    message(STATUS "Found Mac OSX SDK: ${SDKROOT}")
  ENDIF()
  
  set(CMAKE_CXX_FLAGS "-std=c++11 -stdlib=libc++ -isysroot ${SDKROOT}")
  set(CMAKE_C_FLAGS "-isysroot ${SDKROOT}")
ELSE()
  set(CMAKE_CXX_FLAGS "-std=gnu++0x")
ENDIF()

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(public-headers)
include(core)
include(async)
include(objc)
include(LibFindMacros)

IF(APPLE)
  set(additional_includes
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/ctemplate/include
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/icu4c/include
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/libetpan/include
    /usr/include/tidy
    /usr/include/libxml2
    ${SDKROOT}/usr/include
    ${SDKROOT}/usr/include/tidy
    ${SDKROOT}/usr/include/libxml2
  )
  
  set(additional_lib_searchpath
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/ctemplate/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/libetpan/lib
  )
  
  execute_process(COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/../scripts/prepare.sh"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../scripts")
ENDIF()

# detect ctemplate

find_path(CTEMPLATE_INCLUDE_DIR
  NAMES ctemplate/template.h
  PATHS ${additional_includes}
)
find_library(CTEMPLATE_LIBRARY
  NAMES ctemplate
  PATHS ${additional_lib_searchpath}
)

if(NOT CTEMPLATE_INCLUDE_DIR OR NOT CTEMPLATE_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find ctemplate")
else()
  message(STATUS "Found ctemplate")
endif()

# detect icu4c

find_path(ICU4C_INCLUDE_DIR
  NAMES unicode/utf8.h
  PATHS ${additional_includes}
)
if(APPLE)
  find_library(ICU4C_LIBRARY
    NAMES icucore
    PATHS ${additional_lib_searchpath}
  )
else()
  find_library(ICU4C_LIBRARY
    NAMES icuuc
    PATHS ${additional_lib_searchpath}
  )
endif()

if(NOT ICU4C_INCLUDE_DIR OR NOT ICU4C_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find icu4c")
else()
  message(STATUS "Found icu4c")
endif()

# detect libetpan

find_path(LIBETPAN_INCLUDE_DIR
  NAMES libetpan/libetpan.h
  PATHS ${additional_includes}
)
find_library(LIBETPAN_LIBRARY
  NAMES etpan
  PATHS ${additional_lib_searchpath}
)

if(NOT LIBETPAN_INCLUDE_DIR OR NOT LIBETPAN_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find libetpan")
else()
  message(STATUS "Found libetpan")
endif()


# detect tidy

find_path(TIDY_INCLUDE_DIR
  NAMES tidy/tidy.h
  PATHS ${additional_includes}
)
find_library(TIDY_LIBRARY
  NAMES tidy
  PATHS ${additional_lib_searchpath}
)

if(NOT TIDY_INCLUDE_DIR OR NOT TIDY_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find tidy")
else()
  message(STATUS "Found tidy")
endif()


# detect uuid

find_path(UUID_INCLUDE_DIR
  NAMES uuid/uuid.h
  PATHS ${additional_includes}
)

if(NOT UUID_INCLUDE_DIR)
  message(FATAL_ERROR "ERROR: Could not find uuid")
else()
  message(STATUS "Found uuid")
endif()


# detect libxml2

find_path(LIBXML_INCLUDE_DIR
  NAMES libxml/xmlreader.h
  PATHS ${additional_includes}
)
find_library(LIBXML_LIBRARY
  NAMES xml2
  PATHS ${additional_lib_searchpath}
)

if(NOT LIBXML_INCLUDE_DIR OR NOT LIBXML_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find libxml2")
else()
  message(STATUS "Found libxml2")
endif()


# detect zlib

find_path(ZLIB_INCLUDE_DIR
  NAMES zlib.h
  PATHS ${additional_includes}
)
find_library(ZLIB_LIBRARY
  NAMES z
  PATHS ${additional_lib_searchpath}
)

if(NOT ZLIB_INCLUDE_DIR OR NOT ZLIB_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find zlib")
else()
  message(STATUS "Found zlib")
endif()


file(GLOB_RECURSE
  source_files
  *.h
  *.m
  *.c
)

SET_SOURCE_FILES_PROPERTIES(
  ${source_files}
  PROPERTIES LANGUAGE C
)

file(COPY
  ${public_headers}

  DESTINATION
  ${CMAKE_CURRENT_BINARY_DIR}/include/MailCore
)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include
  ${async_includes}
  ${core_includes}
  ${objc_includes}
  ${additional_includes}
  /usr/include/tidy
  /usr/include/libxml2
)

add_library(MailCore
  ${core_files}
  ${async_files}
  ${objc_files}
)
