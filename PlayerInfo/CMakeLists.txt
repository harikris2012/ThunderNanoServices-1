# If not stated otherwise in this file or this component's LICENSE file the
# following copyright and licenses apply:
#
# Copyright 2020 RDK Management
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(PLUGIN_NAME PlayerInfo)
set(MODULE_NAME ${NAMESPACE}${PLUGIN_NAME})

option(DOLBY_SUPPORT "Build the PlayerInfo with dolby support" OFF)

find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

add_library(${MODULE_NAME} SHARED
    PlayerInfo.cpp
    PlayerInfoJsonRpc.cpp
    Module.cpp)

set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins)

find_package(GSTREAMER REQUIRED)

if(DOLBY_SUPPORT)

    target_sources(${MODULE_NAME}
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/Dolby/DolbyOutput.cpp)

    target_include_directories(${MODULE_NAME}
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/Dolby/include)

    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/Dolby)

    target_link_libraries(${MODULE_NAME}
        PRIVATE
            PlayerInfoDolby)
endif()

if (GSTREAMER_FOUND)
    target_sources(${MODULE_NAME}
        PRIVATE
            GStreamer/PlatformImplementation.cpp)

    target_link_libraries(${MODULE_NAME}
        PRIVATE
            ${GSTREAMER_LIBRARIES})

    target_include_directories(${MODULE_NAME}
        PRIVATE
            ${GSTREAMER_INCLUDES})
endif ()

install(TARGETS ${MODULE_NAME} 
    DESTINATION lib/${STORAGE_DIRECTORY}/plugins)

write_config(${PLUGIN_NAME})
