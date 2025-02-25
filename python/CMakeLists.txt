# =============================================================================
# Copyright (c) 2022-2023, NVIDIA CORPORATION.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
# in compliance with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software distributed under the License
# is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
# or implied. See the License for the specific language governing permissions and limitations under
# the License.
# =============================================================================

cmake_minimum_required(VERSION 3.26.4 FATAL_ERROR)

set(kvikio_version 24.02.00)

include(../cpp/cmake/fetch_rapids.cmake)
include(rapids-cpm)
rapids_cpm_init()
include(rapids-cuda)
rapids_cuda_init_architectures(kvikio-python)

project(
  kvikio-python
  VERSION ${kvikio_version}
  LANGUAGES CXX CUDA
)

option(FIND_KVIKIO_CPP
       "Search for existing KVIKIO C++ installations before defaulting to local files" OFF
)

# TODO: Should we symlink FindcuFile.cmake into python/cmake? find cuFile
include(../cpp/cmake/Modules/FindcuFile.cmake)

if(FIND_KVIKIO_CPP)
  find_package(KvikIO ${kvikio_version})
else()
  set(KvikIO_FOUND OFF)
endif()

find_package(CUDAToolkit REQUIRED)

if(NOT KvikIO_FOUND)
  add_subdirectory(../cpp kvikio-cpp)
  set(cython_lib_dir kvikio)
  install(TARGETS kvikio DESTINATION ${cython_lib_dir})
endif()

include(rapids-cython-core)
rapids_cython_init()

add_subdirectory(cmake)
add_subdirectory(kvikio/_lib)
