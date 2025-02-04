cmake_minimum_required(VERSION 3.15...3.20)

project(AppleSiliconCPUdetect LANGUAGES C)

# NOTE: in general MacOS "-framework" is specially handled by link_libraries. link_options does not work.

#----- OPTIONAL demo targets
if(APPLE)
add_executable(as_detect_aspl as_detect_aspl.c)
target_link_libraries(as_detect_aspl PRIVATE "-framework Foundation" "-framework IOKit")

add_executable(as_detect as_detect.c)
target_link_libraries(as_detect PRIVATE "-framework Foundation" "-framework IOKit")
endif(APPLE)
#------ typical CMake use follows

function(count_apple_silicon)

# try_run is like a micro ExternalProject
# built and run at configure time with captured output

try_run(run_code build_ok
 ${CMAKE_CURRENT_BINARY_DIR}/cpu_count
 ${CMAKE_CURRENT_SOURCE_DIR}/as_detect.c
 LINK_LIBRARIES "-framework Foundation" "-framework IOKit"
 COMPILE_OUTPUT_VARIABLE build_out
 RUN_OUTPUT_VARIABLE count_raw
)

if(NOT build_ok)
  message(FATAL_ERROR "failed to build: ${build_out}")
elseif(NOT run_code EQUAL 0)
  message(FATAL_ERROR "failed to run: ${run_code} ${count_raw}")
endif()

message(VERBOSE "${count_raw}")

string(REGEX MATCHALL "(firestorm)" m ${count_raw})

list(LENGTH m Ncpu)

set(Ncpu ${Ncpu} PARENT_SCOPE)

endfunction(count_apple_silicon)


cmake_host_system_information(RESULT sys_info QUERY OS_NAME OS_PLATFORM)
if(CMAKE_SYSTEM_NAME STREQUAL Darwin)
  if(sys_info MATCHES "arm64")
    count_apple_silicon()

    message(STATUS "${Ncpu} Apple Silicon fast CPU cores detected")
  else()
    message(STATUS "Apple Silicon not detected: ${sys_info}")
  endif()
else()
  message(STATUS "Not MacOS")
endif()
