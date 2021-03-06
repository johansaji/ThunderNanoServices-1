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

set(TARGET ${PLATFORM_COMPOSITOR})

message("Setup ${TARGET} v${VERSION}...")

if (PLUGIN_COMPOSITOR_NXSERVER)
add_subdirectory (NexusServer)
endif ()

# add the platformplugin library
add_library(${TARGET} Wayland.cpp)

find_package(WaylandServer REQUIRED)
find_package(WaylandEGL REQUIRED)
find_package(${NAMESPACE}Core REQUIRED)
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(EGL REQUIRED)
find_package(PNG REQUIRED)

if (PLUGIN_COMPOSITOR_NXSERVER)
    target_link_libraries(${TARGET} 
        PRIVATE 
           ${NAMESPACE}nexusserver::${NAMESPACE}nexusserver)
    target_compile_definitions(${TARGET}
        PRIVATE
            ENABLE_NXSERVER=1)
    if (V3D_DRM_DISABLE)
        target_compile_definitions(${TARGET}
            PRIVATE
                V3D_DRM_DISABLE=1)
    endif ()
endif ()

if(EGL_FOUND)
    add_definitions(${EGL_CFLAGS})
endif(EGL_FOUND)

target_link_libraries(${TARGET}
    PRIVATE
        ${NAMESPACE}Core::${NAMESPACE}Core
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions
        WaylandServer::WaylandServer
        WaylandEGL::WaylandEGL
        PNG::PNG
        compositorclient)

target_include_directories(${TARGET}
    PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../Client>
        ${EGL_INCLUDE_DIRS}
)

target_compile_definitions(${TARGET}
    PRIVATE
        ${EGL_DEFINITIONS})

set_target_properties(${TARGET} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES
        FRAMEWORK FALSE
        DEFINE_SYMBOL WPEFRAMEWORK_PLATFORM_PLUGIN
)

find_package(WesterosServer)
if(WESTEROS_SERVER_FOUND)
    target_sources(${TARGET} 
        PRIVATE 
            Westeros.cpp
    )
    target_link_libraries(${TARGET} 
        PRIVATE
            WesterosServer::WesterosServer
    )

    find_package(BCM_HOST QUIET)
    if (NOT BCM_HOST_FOUND)
        target_compile_definitions(${TARGET}
            PRIVATE
                RESOLUTION_SUPPORT=1)
    endif(NOT BCM_HOST_FOUND)

endif(WESTEROS_SERVER_FOUND)

if(PLUGIN_COMPOSITOR_VIRTUALINPUT)
    find_package(${NAMESPACE}VirtualInput REQUIRED)
    target_link_libraries(${TARGET} 
        PRIVATE 
           ${NAMESPACE}VirtualInput::${NAMESPACE}VirtualInput
    )
endif(PLUGIN_COMPOSITOR_VIRTUALINPUT)

install(
    TARGETS ${TARGET}
    DESTINATION ${CMAKE_INSTALL_PREFIX}/share/${NAMESPACE}/Compositor
)

