set(PLUGIN_NAME TextToSpeech)
set(MODULE_NAME ${NAMESPACE}${PLUGIN_NAME})

set(PLUGIN_TEXTTOSPEECH_AUTOSTART "true" CACHE STRING "Automatically start TestToSpeech plugin")
set(PLUGIN_TEXTTOSPEECH_STARTUPORDER "" CACHE STRING "Automatically start TextToSpeech plugin")
set(PLUGIN_TEXTTOSPEECH_MODE "Local" CACHE STRING "Controls if the plugin should run in its own process, in process or remote")

find_package(${NAMESPACE}Plugins REQUIRED)

find_package(WPEFrameworkSecurityUtil)
if(NOT WPEFRAMEWORK_SECURITYUTIL_FOUND)
    add_definitions(-DSECURITY_TOKEN_ENABLED=0)
endif()

set_source_files_properties(
        TextToSpeechImplementation.cpp
        PROPERTIES COMPILE_FLAGS "-fexceptions")

set_source_files_properties(
        TextToSpeechJsonRpc.cpp
        PROPERTIES COMPILE_FLAGS "-fexceptions")

add_library(${MODULE_NAME} SHARED
        Module.cpp
        TextToSpeech.cpp
        TextToSpeechJsonRpc.cpp
        TextToSpeechImplementation.cpp
        impl/TTSManager.cpp
        impl/TTSSpeaker.cpp
        impl/logger.cpp
        impl/TTSDownloader.cpp
        impl/TTSURLConstructer.cpp
        impl/NetworkStatusObserver.cpp
        impl/SatToken.cpp
        impl/RFCURLObserver.cpp
        )
set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

list(APPEND CMAKE_MODULE_PATH
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

find_package(GStreamer REQUIRED)

find_package(Curl)

find_package(${NAMESPACE}Plugins REQUIRED)

find_package(${NAMESPACE}Definitions REQUIRED)

set(AUDIO_CLIENT_LIB "")
if (BUILD_AMLOGIC)
    set(AUDIO_CLIENT_LIB "audio_client")
elseif (BUILD_REALTEK)
    set(AUDIO_CLIENT_LIB "RtkHALMisc")
elseif (BUILD_BROADCOM)
    set(AUDIO_CLIENT_LIB "nxclient") 
endif()

if (RDK_SERVICE_L2_TEST)
    message ("L2 test Enabled")
    find_library(TESTMOCKLIB_LIBRARIES NAMES TestMocklib)
    if (TESTMOCKLIB_LIBRARIES)
        message ("linking mock libraries ${TESTMOCKLIB_LIBRARIES} library")
        target_link_libraries(${MODULE_NAME} PRIVATE ${TESTMOCKLIB_LIBRARIES})
    else (TESTMOCKLIB_LIBRARIES)
        message ("Require ${TESTMOCKLIB_LIBRARIES} library")
    endif (TESTMOCKLIB_LIBRARIES)
endif()

target_include_directories(${MODULE_NAME} PRIVATE ../helpers ${GSTREAMER_INCLUDES})

target_link_libraries(${MODULE_NAME} PRIVATE
    ${NAMESPACE}Plugins::${NAMESPACE}Plugins
    ${NAMESPACE}Definitions::${NAMESPACE}Definitions
    ${CURL_LIBRARY} ${GSTREAMER_LIBRARIES} ${AUDIO_CLIENT_LIB} ${WPEFRAMEWORK_SECURITYUTIL_LIBRARIES}
    )

install(TARGETS ${MODULE_NAME}
        DESTINATION lib/${STORAGE_DIRECTORY}/plugins)

write_config( TextToSpeech )
