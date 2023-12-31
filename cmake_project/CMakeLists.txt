set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.22)

include(support/toolchain.cmake)

# set(CMAKE_C_COMPILER arm-none-eabi-gcc)
# set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
# set(CMAKE_ASM_COMPILER  arm-none-eabi-gcc)
# set(CMAKE_AR arm-none-eabi-ar)
# set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
# set(CMAKE_OBJDUMP arm-none-eabi-objdump)
# set(SIZE arm-none-eabi-size)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(AT32F403ACG C CXX ASM)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 11)

add_compile_options(-mcpu=cortex-m4 -mthumb -mthumb-interwork)
add_compile_options(-ffunction-sections -fdata-sections -fno-common -fmessage-length=0)

add_compile_options($<$<COMPILE_LANGUAGE:ASM>:-x$<SEMICOLON>assembler-with-cpp>)

add_compile_options(-O0 -g)

include_directories(
        user
        board/include 
        at32_lib/cmsis/cm4/core_support 
        at32_lib/cmsis/cm4/device_support
        at32_lib/drivers/inc
        at32_lib/usbd_class/cdc
        at32_lib/usbd_drivers/inc
        )

add_definitions(-DDEBUG -DAT32F403ACGU7 -DAT32_F403A_EVB)

file(GLOB_RECURSE SOURCES "user/*.*" "board/*.*" 
        "at32_lib/cmsis/*.*" "at32_lib/drivers/*.*"
        "at32_lib/usbd_drivers/*.*" "at32_lib/usbd_class/cdc/*.*"
        )

set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/at32_lib/cmsis/cm4/device_support/startup/gcc/linker/AT32F403AxG_FLASH.ld)

add_link_options(-Wl,-gc-sections,--print-memory-usage,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map)
add_link_options(-mcpu=cortex-m4 -mthumb -mthumb-interwork)
#add_link_options(-specs=nano.specs -lc -lgcc)
#add_link_options(-lc -lgcc)
add_link_options(-specs=nosys.specs)
add_link_options(-T ${LINKER_SCRIPT})

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
    COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
    COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")

set(JFLASH_FILE ${PROJECT_BINARY_DIR}/upload.jlink)
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND echo "h" > ${JFLASH_FILE}
    COMMAND echo "loadbin ${BIN_FILE}" >> ${JFLASH_FILE}
    COMMAND echo "r" >> ${JFLASH_FILE}
    COMMAND echo "g" >> ${JFLASH_FILE}
    COMMAND echo "q" >> ${JFLASH_FILE}
    COMMENT "Generating upload.jlink"
)

add_custom_target(upload_jlink
    COMMAND ${JLINK_EXECUTABLE} -device AT32F403ACGU7 -speed 10000 -if swd -autoconnect 1 -NoGui 1 -CommanderScript ${JFLASH_FILE}
    DEPENDS ${PROJECT_NAME}.elf
)
