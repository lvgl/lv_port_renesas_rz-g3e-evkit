cmake_minimum_required(VERSION 3.10)

# if you have ccache installed, clean rebuilds will take only a couple of seconds
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

project(lv_port_renesas_rz_g3e_evkit)

set(LV_BUILD_CONF_PATH ${CMAKE_CURRENT_LIST_DIR}/lv_conf.h CACHE PATH
    "Use this to specify the location of and/or filename of lv_conf.h" FORCE)

add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/lv_port_linux)
