# CMakeLists.txt
#
# Wireshark - Network traffic analyzer
# By Gerald Combs <gerald@wireshark.org>
# Copyright 1998 Gerald Combs
#
# SPDX-License-Identifier: GPL-2.0-or-later
#
cmake_minimum_required (VERSION 2.8.12)
cmake_policy(SET CMP0048 NEW)

project(DLEP DESCRIPTION "Wireshark DLEP Plugin" LANGUAGES C)

find_package(PkgConfig REQUIRED)

pkg_check_modules(WIRESHARK REQUIRED wireshark>=2.5)

pkg_get_variable(WIRESHARK_PLUGIN_DIR wireshark plugindir)

set(PLUGIN_INSTALL_LIBDIR "${WIRESHARK_PLUGIN_DIR}/epan")

include(CMakePushCheckState)
include(CheckFunctionExists)


cmake_push_check_state()
set(CMAKE_REQUIRED_LIBRARIES ${WIRESHARK_LDFLAGS})
check_function_exists("proto_register_plugin" HAVE_PROTO_REGISTER_PLUGIN)
cmake_pop_check_state()
if(NOT HAVE_PROTO_REGISTER_PLUGIN)
	message(FATAL_ERROR "Wireshark was compiled without support for plugins")
endif()

set(CMAKE_C_VISIBILITY_PRESET hidden)
if (CMAKE_COMPILER_IS_GNUCC)
    set(CMAKE_C_FLAGS  "-Wall -Wextra ${CMAKE_C_FLAGS}")
endif()

add_definitions(-DHAVE_PLUGINS)

add_library(dlep MODULE packet-dlep.c)
set_target_properties(dlep PROPERTIES PREFIX "" DEFINE_SYMBOL "")
target_link_libraries(dlep ${WIRESHARK_LDFLAGS})
target_compile_options(dlep PUBLIC ${WIRESHARK_CFLAGS})

install(TARGETS dlep
	LIBRARY DESTINATION "${PLUGIN_INSTALL_LIBDIR}" NAMELINK_SKIP
)

#
# Editor modelines  -  http://www.wireshark.org/tools/modelines.html
#
# Local variables:
# c-basic-offset: 8
# tab-width: 8
# indent-tabs-mode: t
# End:
#
# vi: set shiftwidth=8 tabstop=8 noexpandtab:
# :indentSize=8:tabSize=8:noTabs=false:
#
