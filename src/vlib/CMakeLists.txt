# Copyright (c) 2018 Cisco and/or its affiliates.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

option(VPP_BUFFER_FAULT_INJECTOR "Include the buffer fault injector" OFF)

##############################################################################
# Generate vlib/config.h
##############################################################################
if(VPP_BUFFER_FAULT_INJECTOR)
  set(BUFFER_ALLOC_FAULT_INJECTOR 1 CACHE STRING "fault injector on")
else()
  set(BUFFER_ALLOC_FAULT_INJECTOR 0 CACHE STRING "fault injector off")
endif()

set(PRE_DATA_SIZE 128 CACHE STRING "Buffer headroom size.")
configure_file(
  ${CMAKE_SOURCE_DIR}/vlib/config.h.in
  ${CMAKE_BINARY_DIR}/vlib/config.h
)
install(
  FILES ${CMAKE_BINARY_DIR}/vlib/config.h
  DESTINATION include/vlib
  COMPONENT vpp-dev
)

##############################################################################
# Find lib and include files
##############################################################################
message(STATUS "Looking for libuuid")
find_path(UUID_INCLUDE_DIR NAMES uuid/uuid.h)
find_library(UUID_LIB NAMES uuid)

if(UUID_INCLUDE_DIR AND UUID_LIB)
  include_directories(${UUID_INCLUDE_DIR})
  set(VMBUS_SOURCE linux/vmbus.c)
  set(VMBUS_LIBS uuid)
  message(STATUS "Found uuid in ${UUID_INCLUDE_DIR}")
else()
  message(WARNING "-- libuuid not found - vmbus support disabled")
endif()

##############################################################################
# vlib shared library
##############################################################################
add_vpp_library(vlib
  SOURCES
  buffer.c
  cli.c
  counter.c
  drop.c
  error.c
  format.c
  handoff_trace.c
  init.c
  linux/pci.c
  linux/vfio.c
  log.c
  main.c
  node.c
  node_cli.c
  node_format.c
  node_init.c
  pci/pci.c
  pci/pci_types_api.c
  physmem.c
  punt.c
  punt_node.c
  threads.c
  threads_cli.c
  trace.c
  unix/cli.c
  unix/input.c
  unix/main.c
  unix/plugin.c
  unix/util.c
  vmbus/vmbus.c
  ${VMBUS_SOURCE}

  MULTIARCH_SOURCES
  drop.c
  punt_node.c
  node_init.c

  INSTALL_HEADERS
  buffer_funcs.h
  buffer.h
  buffer_node.h
  cli.h
  counter.h
  counter_types.h
  defs.h
  error_funcs.h
  error.h
  format_funcs.h
  global_funcs.h
  init.h
  linux/vfio.h
  log.h
  main.h
  node_funcs.h
  node.h
  pci/pci_config.h
  pci/pci.h
  pci/pci_types_api.h
  physmem_funcs.h
  physmem.h
  punt.h
  threads.h
  trace_funcs.h
  trace.h
  unix/mc_socket.h
  unix/plugin.h
  unix/unix.h
  vlib.h
  vmbus/vmbus.h

  API_FILES
  pci/pci_types.api

  LINK_LIBRARIES vppinfra svm ${VMBUS_LIBS} ${CMAKE_DL_LIBS}

  DEPENDS api_headers
)
