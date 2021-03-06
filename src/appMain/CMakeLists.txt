# Copyright (c) 2014, Ford Motor Company
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following
# disclaimer in the documentation and/or other materials provided with the
# distribution.
#
# Neither the name of the Ford Motor Company nor the names of its contributors
# may be used to endorse or promote products derived from this software
# without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

IF (HMIADAPTER STREQUAL "messagebroker")
set (BROKER_LIBRARIES
  MessageBrokerClient
  MessageBrokerServer
)
ENDIF ()

cmake_policy(PUSH)
# make link_directories() treat paths relative to the source dir
# info: cmake --help-policy CMP0015
cmake_policy(SET CMP0015 NEW)
link_directories(${LIBUSB_LIBS_DIRECTORY})
cmake_policy(POP)

if (EXTENDED_MEDIA_MODE)
set(default_media_inc
${GSTREAMER_gst_INCLUDE_DIR}
${GSTREAMER_gstconfig_INCLUDE_DIR}
)
else(EXTENDED_MEDIA_MODE)
set(default_media_inc
)
endif()

if (TELEMETRY_MONITOR)
    set(TELEMETRY_MONITOR_LIB
        TelemetryMonitor)
endif()

link_directories(
  ${OPENSSL_LIB_DIRECTORY}
)

set(LIBRARIES
  MediaManager
  ApplicationManager
  MessageHelper
  HMI_API
  MOBILE_API
  v4_protocol_v1_2_no_extra
  SmartObjects
  ${TELEMETRY_MONITOR_LIB}
  formatters
  ProtocolLibrary
  ProtocolHandler
  connectionHandler
  TransportManager
  ${SecurityManagerLibrary}
  HMIMessageHandler
  MessageBroker
  ${BROKER_LIBRARIES}
  Utils
  jsoncpp
  ConfigProfile
  Resumption
)

if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
  list(APPEND LIBRARIES pthread)
  list(APPEND LIBRARIES dl)
endif()

if (BUILD_BT_SUPPORT)
  list(APPEND LIBRARIES bluetooth)
endif()
if (BUILD_USB_SUPPORT)
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
  list(APPEND LIBRARIES Libusb-1.0.16)
endif()
endif()

if(ENABLE_LOG)
  list(APPEND LIBRARIES log4cxx -L${LOG4CXX_LIBS_DIRECTORY})
  list(APPEND LIBRARIES apr-1 -L${APR_LIBS_DIRECTORY})
  list(APPEND LIBRARIES aprutil-1 -L${APR_UTIL_LIBS_DIRECTORY})
  list(APPEND LIBRARIES expat -L${EXPAT_LIBS_DIRECTORY})
endif()

include_directories (
  ./
  ${COMPONENTS_DIR}/protocol_handler/include/
  ${JSONCPP_INCLUDE_DIRECTORY}
  ${COMPONENTS_DIR}/application_manager/include
  ${COMPONENTS_DIR}/formatters/include
  ${COMPONENTS_DIR}/transport_manager/include
  ${COMPONENTS_DIR}/security_manager/include
  ${SecurityManagerIncludeDir}
  ${COMPONENTS_DIR}/security_manager/include
  ${COMPONENTS_DIR}/config_profile/include
  ${COMPONENTS_DIR}/include/
  ${COMPONENTS_DIR}/utils/include/
  ${COMPONENTS_DIR}/connection_handler/include/
  ${COMPONENTS_DIR}/hmi_message_handler/include
  ${COMPONENTS_DIR}/request_watchdog/include
  ${COMPONENTS_DIR}/smart_objects/include/
  ${COMPONENTS_DIR}/media_manager/include/
  ${COMPONENTS_DIR}/telemetry_monitor/include
  ${COMPONENTS_DIR}/policy/include/
  ${COMPONENTS_DIR}/resumption/include/
  ${MESSAGE_BROKER_INCLUDE_DIRECTORY}
  ${ENCRYPTION_INCLUDE_DIRECTORY}
  ${COMPONENTS_DIR}
  ${CMAKE_BINARY_DIR}/src/components/
  ${COMPONENTS_DIR}/dbus/include/
  ${CMAKE_SOURCE_DIR}
  ${default_media_inc}
  ${LOG4CXX_INCLUDE_DIRECTORY}
  ${CMAKE_SOURCE_DIR}/src/3rd_party/dbus-1.7.8
  ${CMAKE_SOURCE_DIR}/src/3rd_party/dbus-1.7.8/dbus/
)

set (SOURCES
  main.cc
  life_cycle.cc
  signal_handlers.cc
)

if( NOT CMAKE_BUILD_TYPE )
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build. Options are: None, Debug, Release, RelWithDebInfo, MinSizeRel." FORCE)
endif()

add_executable(${PROJECT} ${SOURCES})
target_link_libraries(${PROJECT} ${LIBRARIES})


file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/log4cxx.properties DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/audio.8bit.wav DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test.txt DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/smartDeviceLink.ini DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/hmi_capabilities.json DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/sdl_preloaded_pt.json DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
if (CMAKE_SYSTEM_NAME STREQUAL "QNX")
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/init_policy.sh DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
endif ()

# Replace commit in ini file
set(GITCOMMIT "")
if(EXISTS ${CMAKE_SOURCE_DIR}/.git)
  find_package(Git)
  if(GIT_FOUND)
    execute_process(
      COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
      WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
      OUTPUT_VARIABLE GITCOMMIT ERROR_QUIET
      OUTPUT_STRIP_TRAILING_WHITESPACE)
    set(INI_FILE ${CMAKE_CURRENT_BINARY_DIR}/smartDeviceLink.ini)
    if(EXISTS ${INI_FILE})
      file(READ ${INI_FILE} FILE_CONTENT)
      set(LINE SDLVersion)
      set(SEARCH_REGEX "${LINE}([^/\r/\n]+)")
      string(REGEX REPLACE "${SEARCH_REGEX}" "${LINE} = ${GITCOMMIT}" 
             MODIFIED_FILE_CONTENT "${FILE_CONTENT}")
      file(WRITE "${INI_FILE}" "${MODIFIED_FILE_CONTENT}")
    endif()
  endif(GIT_FOUND)
endif()

if (${QT_HMI})
  set(main_qml "hmi/MainWindow.qml")
  set(plugins_dir "hmi/plugins")

  if (CMAKE_SYSTEM_NAME STREQUAL "QNX")
    set(start_command "./start-qml.sh")
    set(command_arguments "${main_qml} ${plugins_dir}")
  else ()
    execute_process(
      COMMAND ${CMAKE_SOURCE_DIR}/FindQt.sh -v ${qt_version} -b qmlscene
      OUTPUT_VARIABLE start_command
    )
    set(command_arguments "${main_qml} -I ${plugins_dir}")
  endif ()

  file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/start_hmi.sh
    "#!/bin/sh\n${start_command} ${command_arguments}\n")
endif ()

# Install rules
install(TARGETS ${PROJECT} DESTINATION bin)
install(
  FILES log4cxx.properties audio.8bit.wav test.txt smartDeviceLink.ini
    hmi_capabilities.json sdl_preloaded_pt.json
    ${CMAKE_SOURCE_DIR}/mycert.pem ${CMAKE_SOURCE_DIR}/mykey.pem
  DESTINATION bin
)

if (${WEB_HMI})
  if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    install( DIRECTORY ${CMAKE_HOME_DIRECTORY}/src/components/HMI/ DESTINATION bin/HMI)
  endif ()
elseif (${QT_HMI})
  if (CMAKE_SYSTEM_NAME STREQUAL "QNX")
    install(FILES start-qml.sh DESTINATION bin
      PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ
        GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
  endif ()
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/start_hmi.sh DESTINATION bin
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ
      GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
endif ()
if (CMAKE_SYSTEM_NAME STREQUAL "QNX")
  install(FILES init_policy.sh DESTINATION bin
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ
      GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
endif ()
