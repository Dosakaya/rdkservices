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

cmake_minimum_required(VERSION 3.14)

project(sqlitel1test)

set(CMAKE_CXX_STANDARD 11)

include(FetchContent)
FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG main
)
FetchContent_MakeAvailable(googletest)

find_package(WPEFramework NAMES WPEFramework Thunder)
find_package(${NAMESPACE}Plugins REQUIRED)

add_executable(${PROJECT_NAME}
        ../../Module.cpp
        Store2Test.cpp
)

target_link_libraries(${PROJECT_NAME} PRIVATE
        gmock_main
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
)

find_package(PkgConfig REQUIRED)
pkg_search_module(SQLITE REQUIRED sqlite3)
target_link_libraries(${PROJECT_NAME} PRIVATE ${SQLITE_LIBRARIES})

install(TARGETS ${PROJECT_NAME} DESTINATION bin)
