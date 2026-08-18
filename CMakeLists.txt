CMAKE_MINIMUM_REQUIRED(VERSION 3.14)

PROJECT("xva-img")

SET(CMAKE_CXX_FLAGS_RELEASE "-Wall -Wextra -Wvla")
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

SET(CMAKE_CXX_STANDARD 11)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)

# If on Windows, we can append C:/Tools/mingw64/opt to CMAKE_PREFIX_PATH to find OpenSSL
IF(WIN32)
  LIST(APPEND CMAKE_PREFIX_PATH "C:/Tools/mingw64/opt")
ENDIF()

FIND_PACKAGE(OpenSSL REQUIRED)

# For xxHash, we can fetch its source and use it as a header-only library (using XXH_INLINE_ALL).
# This avoids running xxHash's internal CMakeLists.txt, which is incompatible with modern CMake versions.
INCLUDE(FetchContent)
FetchContent_Declare(
  xxHash
  GIT_REPOSITORY https://github.com/Cyan4973/xxHash.git
  GIT_TAG        v0.8.2
)

# Populate the source code without invoking the project's CMakeLists.txt
FetchContent_GetProperties(xxHash)
IF(NOT xxhash_POPULATED)
  FetchContent_Populate(xxHash)
ENDIF()

SET(TARGET_NAME "xva-img")
FILE(GLOB_RECURSE SOURCE_FILES src/*.cpp)
ADD_EXECUTABLE(${TARGET_NAME} ${SOURCE_FILES})

# Include the project's src directory and the fetched xxHash source directory
# Note: FetchContent lower-cases the variable prefix (xxhash_SOURCE_DIR)
TARGET_INCLUDE_DIRECTORIES(${TARGET_NAME} PRIVATE src ${xxhash_SOURCE_DIR})

# Use xxHash as header-only
TARGET_COMPILE_DEFINITIONS(${TARGET_NAME} PRIVATE XXH_INLINE_ALL)

TARGET_LINK_LIBRARIES(${TARGET_NAME}
	OpenSSL::Crypto
)

# On Windows, we might need ws2_32 for some sockets/networking or other libraries
IF(WIN32)
	TARGET_LINK_LIBRARIES(${TARGET_NAME} ws2_32)
ENDIF()

INSTALL(TARGETS ${TARGET_NAME}
	DESTINATION bin)
