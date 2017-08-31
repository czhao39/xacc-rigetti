#***********************************************************************************
# Copyright (c) 2016, UT-Battelle
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#   * Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in the
#     documentation and/or other materials provided with the distribution.
#   * Neither the name of the xacc nor the
#     names of its contributors may be used to endorse or promote products
#     derived from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
# Contributors:
#   Initial API and implementation - Alex McCaskey
#
#**********************************************************************************/

find_package(OpenSSL)
find_path(CPPREST_INCLUDE cpprest/http_client.h)
find_library(CPPREST_LIB NAMES cpprest_2_9d cpprest_2_9 cpprestd cpprest)

if (NOT CPPREST_LIB)
	message (STATUS "COULD NOT FIND CPPRESTLIB")
endif()
if (OPENSSL_FOUND AND CPPREST_LIB)

   include_directories(${CPPREST_INCLUDE})

   set (PACKAGE_NAME "Rigetti XACC Accelerator")
   set (PACKAGE_DESCIPTION "Rigetti XACC Accelerator Framework")
   set (LIBRARY_NAME xacc-rigetti-accelerator)

   file (GLOB_RECURSE HEADERS *.hpp)
   file (GLOB SRC *.cpp)

   # Set up dependencies to resources to track changes
   usFunctionGetResourceSource(TARGET ${LIBRARY_NAME} OUT SRC)
   # Generate bundle initialization code
   usFunctionGenerateBundleInit(TARGET ${LIBRARY_NAME} OUT SRC)

   add_library(${LIBRARY_NAME} SHARED ${SRC})

   set(_bundle_name xacc_rigetti_accelerator) 

   set_target_properties(${LIBRARY_NAME} PROPERTIES
     # This is required for every bundle
     COMPILE_DEFINITIONS US_BUNDLE_NAME=${_bundle_name}
     # This is for convenience, used by other CMake functions
     US_BUNDLE_NAME ${_bundle_name}
     )

   if(BUILD_SHARED_LIBS)
     set_target_properties(${LIBRARY_NAME} PROPERTIES
       CXX_VISIBILITY_PRESET hidden
       VISIBILITY_INLINES_HIDDEN 1
       )
   endif()

   # Embed meta-data from a manifest.json file
   usFunctionEmbedResources(TARGET ${LIBRARY_NAME}
     WORKING_DIRECTORY
       ${CMAKE_CURRENT_SOURCE_DIR}
     FILES
       manifest.json
     )

   target_link_libraries(${LIBRARY_NAME} ${Boost_LIBRARIES} ${OPENSSL_LIBRARIES} ${CPPREST_LIB} ${XACC_LIBRARIES})

   install(TARGETS ${LIBRARY_NAME} DESTINATION ${XACC_ROOT}/lib/plugins/accelerators)
   
   file (GLOB test_files ../tests/RigettiAcceleratorTester.cpp)
   add_tests("${test_files}" "${CMAKE_CURRENT_SOURCE_DIR}" "${Boost_LIBRARIES};xacc-rigetti-accelerator")
   
else()
	message(STATUS "OpenSSL or CppRestSDK not found, cannot include Rigetti Accelerator plugin.")
endif()