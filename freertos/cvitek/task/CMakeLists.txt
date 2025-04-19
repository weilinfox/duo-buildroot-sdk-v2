cmake_minimum_required(VERSION 3.16.5)

project(arch C ASM)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

# to build task/ without build the whole firmware
# -DRUN_TYPE=CVIRTOS
# -DRUN_TYPE=RADIO_DEMO
set(TOP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../")
set(BUILD_ENV_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../build/")
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/../scripts/toolchain-riscv64-elf.cmake")
set(RUN_ARCH "riscv64")
# set(CHIP "cv180x")

# see https://cmake.org/cmake/help/latest/policy/CMP0175.html
cmake_policy(SET CMP0175 OLD)

set(CMAKE_INSTALL_PREFIX ${TOP_DIR}/install)
set(CMAKE_INSTALL_INC_PREFIX ${TOP_DIR}/install/include)

set(SAFETY_FLAGS "-Wall -Wextra -fno-strict-aliasing -static --specs=nosys.specs")
#set(SAFETY_FLAGS "-Werror -Wall -Wextra -fno-strict-aliasing")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${SAFETY_FLAGS} -D${RUN_TYPE}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${SAFETY_FLAGS}")

include(${TOP_DIR}/scripts/ParseConfiguration.cmake)
ParseConfiguration("${BUILD_ENV_PATH}/.config")

if (RUN_TYPE STREQUAL "BLINKY_DEMO")
    add_subdirectory(demo/Blinky_Demo)
elseif (RUN_TYPE STREQUAL "FULL_DEMO")
    add_subdirectory(demo/Full_Demo)
elseif (RUN_TYPE STREQUAL "POSIX_DEMO")
    add_subdirectory(demo/POSIX_Demo)
elseif (RUN_TYPE STREQUAL "CVIRTOS")
    add_subdirectory(comm)
    add_subdirectory(rgn)
elseif (RUN_TYPE STREQUAL "RADIO_DEMO")
    add_subdirectory(radio)

if(CONFIG_BOARD STREQUAL "cv181x_fpga" OR CONFIG_BOARD STREQUAL "cv181x_fpga_c906")
	add_compile_definitions(FPGA_PORTING)
endif()

endif()
add_subdirectory(main)

add_custom_target(cvirtos.bin
    DEPENDS ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.elf
)
add_custom_command(TARGET cvirtos.bin
    COMMAND echo "Build cvirtos.bin and it depends on ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.elf"
    POST_BUILD
    DEPENDS ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.elf
    COMMAND ${CMAKE_OBJCOPY} ARGS ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.elf -O binary ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.bin
    COMMAND ${CMAKE_OBJDUMP} ARGS -dS ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.elf > ${CMAKE_INSTALL_PREFIX}/bin/cvirtos.dis
    VERBATIM
)
