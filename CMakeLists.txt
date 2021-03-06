cmake_minimum_required(VERSION 2.6)

project (ympd)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_GENERATOR "DEB;RPM;TGZ")
set(CPACK_SOURCE_GENERATOR "TBZ2")
set(DEBIAN_PACKAGE_SECTION "web")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "MPD web client based on Websockets and Bootstrap")
set(CPACK_PACKAGE_CONTACT "Andrew Karpow <andy@ympd.org>")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "andy@ndyk.de")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libssl1.0.0,libmpdclient2")

option(WITH_STATIC_WEBSOCKETS "Build with static libwebsockets library" ON)
option(WITH_MPD_HOST_CHANGE "Let users of the web frontend change the MPD Host" ON)

find_package(LibMPDClient REQUIRED)
find_package(LibWebSockets REQUIRED)
if(WITH_STATIC_WEBSOCKETS)
    find_package(OpenSSL REQUIRED)
    find_package(ZLIB REQUIRED)
endif()

configure_file(${PROJECT_SOURCE_DIR}/src/config.h.in
    ${PROJECT_BINARY_DIR}/config.h)
include_directories(${PROJECT_BINARY_DIR} ${LIBWEBSOCKETS_INCLUDE_DIR})

include(CheckCSourceCompiles)
include(CPack)

set(CMAKE_C_FLAGS "-std=gnu99 -Wall") 
set(CMAKE_C_FLAGS_DEBUG "-ggdb -pedantic")

set(SOURCES
    src/ympd.c
    src/http_server.c
    src/mpd_client.c
)

add_executable(ympd ${SOURCES})

# TODO: use generator expressions introduced to CMake 2.8.12, too fresh yet
if(WITH_STATIC_WEBSOCKETS)
    find_library(LIBWEBSOCKETS_LIBRARY_STATIC libwebsockets.a)
    target_link_libraries(ympd ${LIBMPDCLIENT_LIBRARY}
        ${LIBWEBSOCKETS_LIBRARY_STATIC} ${OPENSSL_LIBRARIES} ${ZLIB_LIBRARIES})
else()
    target_link_libraries(ympd ${LIBMPDCLIENT_LIBRARY}
        ${LIBWEBSOCKETS_LIBRARIES})
endif()


install(TARGETS ympd DESTINATION bin)
install(FILES ympd.1 DESTINATION ${CMAKE_INSTALL_PREFIX}/share/man/man1)
install(DIRECTORY htdocs DESTINATION share/${PROJECT_NAME})
