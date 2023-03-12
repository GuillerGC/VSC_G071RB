cmake_minimum_required(VERSION 3.23.1)

# Optional: print out extra messages to see what is going on. Comment it to have less verbose messages
set(CMAKE_VERBOSE_MAKEFILE ON)

# Path to toolchain file. This one has to be before 'project()' below
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/Buildtools.cmake)

# Setup project, output and linker file
project(VSC_G071RB)
set(EXECUTABLE ${PROJECT_NAME}.elf)
set(LINKER_FILE ${CMAKE_SOURCE_DIR}/Device/STM32G071RBTX_FLASH.ld)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Optional: issue a message to be sure it uses the correct toolchain file.
message(STATUS "CMAKE_TOOLCHAIN_FILE is: ${CMAKE_TOOLCHAIN_FILE}")

set( SEGGER_RTT_SRCS ${CMAKE_SOURCE_DIR}/Lib/SEGGER_RTT )

# List of source files
set(SRC_FILES
        main/main.c

        Device/stm32g0xx_it.c
        Device/system_stm32g0xx.c
        Device/startup_stm32g071xx.s
        Device/stm32g0xx_hal_msp.c
        
        Drivers/Src/stm32g0xx_hal.c 
        Drivers/Src/stm32g0xx_hal_rcc.c 
        Drivers/Src/stm32g0xx_hal_rcc_ex.c 
        Drivers/Src/stm32g0xx_hal_flash.c
        Drivers/Src/stm32g0xx_hal_flash_ex.c
        Drivers/Src/stm32g0xx_hal_dma.c
        Drivers/Src/stm32g0xx_hal_dma_ex.c
        Drivers/Src/stm32g0xx_hal_gpio.c 
        Drivers/Src/stm32g0xx_hal_cortex.c 
        Drivers/Src/stm32g0xx_hal_exti.c 
        Drivers/Src/stm32g0xx_hal_tim.c 
        Drivers/Src/stm32g0xx_hal_tim_ex.c 
        Drivers/Src/stm32g0xx_hal_pwr.c
        Drivers/Src/stm32g0xx_hal_pwr_ex.c
        )

set( SRC_RTT_FILES
     ${SEGGER_RTT_SRCS}/SEGGER_RTT_ASM_ARMv7M.S
     ${SEGGER_RTT_SRCS}/SEGGER_RTT_printf.c 
     ${SEGGER_RTT_SRCS}/SEGGER_RTT_Syscalls_GCC.c 
     ${SEGGER_RTT_SRCS}/SEGGER_RTT.c )

# Build the executable based on the source files
add_executable(${EXECUTABLE} ${SRC_FILES} ${SRC_RTT_FILES} )

# List of compiler defines, prefix with -D compiler option
target_compile_definitions(${EXECUTABLE} PRIVATE
        -DSTM32G071xx
        -DUSE_HAL_DRIVER
        )

# List of includ directories
target_include_directories(${EXECUTABLE} PRIVATE
        main
        CMSIS/Include
        CMSIS/Device/ST/STM32G0xx
        Device
        Drivers/Inc/
        Drivers/Inc/Legacy
        ${SEGGER_RTT_SRCS}
        )

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m0
        -mthumb
        -mfloat-abi=soft
        -fdata-sections
        -ffunction-sections
        
        -Wall
        -O0
        -Og
        -gdwarf-2
        )

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
        -T${LINKER_FILE}
        -mcpu=cortex-m0
        -mthumb
        -mfloat-abi=soft
        --specs=rdimon.specs
        --specs=nano.specs
        -lc
        -lm
        -lnosys
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        -Xlinker -print-memory-usage -Xlinker
        )

# Optional: Print executable size as part of the post build process
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE})

# Optional: Create hex, bin and S-Record files after the build
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O srec --srec-len=64 ${EXECUTABLE} ${PROJECT_NAME}.s19
        COMMAND ${CMAKE_OBJCOPY} -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)