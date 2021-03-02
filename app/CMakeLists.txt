###########################################################################
# Copyright 2018 Konsulko Group
# Copyright 2020 Collabora, Ltd.
#
# Author: Scott Murray <scott.murray@konsulko.com>
# Author: Marius Vlad <marius.vlad@collabora.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
###########################################################################

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(PkgConfig REQUIRED)
find_program(WAYLAND_SCANNER_EXECUTABLE NAMES wayland-scanner)

PROJECT_TARGET_ADD(camera-gstreamer)

add_custom_command(
	OUTPUT  agl-shell-desktop-client-protocol.h
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} client-header
	< ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
	> ${CMAKE_SOURCE_DIR}/app/agl-shell-desktop-client-protocol.h
	DEPENDS ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
)


add_custom_command(
	OUTPUT  ${CMAKE_BINARY_DIR}/app/agl-shell-desktop-client-protocol.h
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} client-header
	< ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
	> ${CMAKE_SOURCE_DIR}/app/agl-shell-desktop-client-protocol.h
	DEPENDS ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
)

add_custom_command(
	OUTPUT  agl-shell-desktop-protocol.c
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} code
	< ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
	> ${CMAKE_BINARY_DIR}/app/agl-shell-desktop-protocol.c
	DEPENDS ${CMAKE_SOURCE_DIR}/app/protocol/agl-shell-desktop.xml
)


pkg_check_modules(GSTREAMER REQUIRED gstreamer-1.0)
pkg_check_modules(GSTREAMER_PLUGINS_BASE REQUIRED gstreamer-plugins-base-1.0)
pkg_check_modules(GSTREAMER_VIDEO REQUIRED gstreamer-video-1.0)
pkg_check_modules(GSTREAMER_PLUGINS_BAD REQUIRED gstreamer-plugins-bad-1.0)

pkg_check_modules(WAYLAND_CLIENT REQUIRED wayland-client)
pkg_check_modules(WAYLAND_PROTOCOLS REQUIRED wayland-protocols>=1.18)
pkg_get_variable(WAYLAND_PROTOCOLS_BASE wayland-protocols pkgdatadir)

add_custom_command(
	OUTPUT  xdg-shell-client-protocol.h
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} client-header
	< ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
	> ${CMAKE_SOURCE_DIR}/app/xdg-shell-client-protocol.h
	DEPENDS ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
)


add_custom_command(
	OUTPUT  ${CMAKE_BINARY_DIR}/app/xdg-shell-client-protocol.h
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} client-header
	< ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
	> ${CMAKE_SOURCE_DIR}/app/xdg-shell-client-protocol.h
	DEPENDS ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
)

add_custom_command(
	OUTPUT  xdg-shell-protocol.c
	COMMAND ${WAYLAND_SCANNER_EXECUTABLE} code
	< ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
	> ${CMAKE_BINARY_DIR}/app/xdg-shell-protocol.c
	DEPENDS ${WAYLAND_PROTOCOLS_BASE}/stable/xdg-shell/xdg-shell.xml
)

add_executable(${TARGET_NAME}
	main.cpp
	utils.h
	utils.cpp
	agl-shell-desktop-protocol.c
	agl-shell-desktop-client-protocol.h
	xdg-shell-protocol.c
	xdg-shell-client-protocol.h
	${RESOURCES}
)

include_directories(
	"${GSTREAMER_INCLUDE_DIRS}"
	"${GSTREAMER_PLUGINS_BASE_INCLUDE_DIRS}"
	"${GSTREAMER_PLUGINS_BAD_INCLUDE_DIRS}"
	"${GSTREAMER_VIDEO_INCLUDE_DIRS}"
)

set_target_properties(${TARGET_NAME} PROPERTIES
	LABELS "EXECUTABLE"
	PREFIX ""
	COMPILE_FLAGS "${EXTRAS_CFLAGS} -DFOR_AFB_BINDING"
	LINK_FLAGS "${BINDINGS_LINK_FLAG}"
	LINK_LIBRARIES "${EXTRAS_LIBRARIES}"
	OUTPUT_NAME "${TARGET_NAME}"
)

target_link_libraries(${TARGET_NAME}
	${GSTREAMER_LIBRARIES}
	"${GSTREAMER_PLUGINS_BASE_LIBRARIES}"
	"${GSTREAMER_PLUGINS_BAD_LIBRARIES}"
	"${GSTREAMER_VIDEO_LIBRARIES}"
	${WAYLAND_CLIENT_LIBRARIES}
	-lgstwayland-1.0
)
