#       ___  ___
#      / / \/ / \
#     / /  / /__/
#    / /__/\___/
#   /_/ plating
#  electronic
#
#  Author: Adrian Antonana <adrian.antonana@plating.de>, March 2021
#

cmake_minimum_required(VERSION 3.13)

project(input-utils VERSION 1.0)

include(GNUInstallDirs)

#=======================================================================
# generate headers
#=======================================================================

set(TYPE_LIST EV REL ABS MSC LED SND REP KEY BTN SW BUS)
set(GENERATED_HEADERS EV.h REL.h ABS.h MSC.h LED.h SND.h REP.h KEY.h BTN.h SW.h BUS.h)

foreach(TYPE IN LISTS TYPE_LIST)

    add_custom_command(
        OUTPUT ${TYPE}.h
        COMMAND ${CMAKE_SOURCE_DIR}/name.sh ${TYPE} > ${CMAKE_CURRENT_BINARY_DIR}/${TYPE}.h
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )

endforeach(TYPE)

#=======================================================================
# libs
#=======================================================================

add_library(input OBJECT
    input.c
    ${GENERATED_HEADERS}
)

target_include_directories(input PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}
)

add_library(tcp OBJECT
    tcp.c
)

#=======================================================================
# utils
#=======================================================================

add_executable(lsinput
    lsinput.c
    $<TARGET_OBJECTS:input>
)

add_executable(input-events
    input-events.c
    $<TARGET_OBJECTS:input>
)

add_executable(input-kbd
    input-kbd.c
    $<TARGET_OBJECTS:input>
)

add_executable(input-send
    input-send.c
    $<TARGET_OBJECTS:input>
    $<TARGET_OBJECTS:tcp>
)

add_executable(input-recv
    input-recv.c
    $<TARGET_OBJECTS:input>
    $<TARGET_OBJECTS:tcp>
)

#=======================================================================
# Install targets and files
#=======================================================================

install(TARGETS lsinput input-events input-kbd input-send input-recv
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
