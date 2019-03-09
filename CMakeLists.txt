cmake_minimum_required(VERSION 3.10)

project(usockets)

include(CheckIPOSupported)
check_ipo_supported(RESULT ipo_supported OUTPUT error)
if(result)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
else()
  message(WARNING "IPO is not supported: ${error}")
endif()

option(USE_UV "use lib_uv binding" OFF)

option(USE_SSL "use ssl" OFF)

file(GLOB_RECURSE SRC_FILES ./src/*.c)

add_compile_options($<$<COMPILE_LANGUAGE:C>:--std=c11>)
add_compile_options($<$<COMPILE_LANGUAGE:C>:-Wall>)

include_directories(./src)

set(LINK_LIBS)
if(USE_UV)
  set(LINK_LIBS ${LINK_LIBS} uv)
  add_definitions(-DLIBUS_USE_LIBUV)
endif(USE_UV)

if(USE_SSL)
  set(LINK_LIBS ${LINK_LIBS} ssl crypto)
else(USE_SSL)
  add_definitions(-DLIBUS_NO_SSL)
endif(USE_SSL)

add_library(${PROJECT_NAME} ${SRC_FILES})
#
file(GLOB OUT_FILES ./examples/*.c)
FOREACH(file ${OUT_FILES})
  get_filename_component(base ${file} NAME)
  get_filename_component(base ${base} NAME_WE)
  add_executable(${base} ${file})
  target_link_libraries(${base} ${PROJECT_NAME} ${LINK_LIBS})
ENDFOREACH()
