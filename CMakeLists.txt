cmake_minimum_required(VERSION 3.15.3)

project(stm32c0)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(STM32CUBEMX_GENERATED_FILES
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_gpio_ex.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_tim.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_tim_ex.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_uart.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_rcc.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_rcc_ex.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_gpio.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_dma.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_cortex.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_pwr.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_flash.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_flash_ex.c
	BSP/Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_exti.c
	
	BSP/Core/Src/main.c
	BSP/Core/Src/stm32f1xx_it.c
	BSP/Core/Src/stm32f1xx_hal_msp.c
	BSP/Core/Src/system_stm32f1xx.c
	
	BSP/startup_stm32f103xb.s)
	
set(EXECUTABLE ${PROJECT_NAME}.out)

add_executable(${EXECUTABLE} ${STM32CUBEMX_GENERATED_FILES})

target_compile_definitions(${EXECUTABLE} PRIVATE
        -DUSE_HAL_DRIVER
        -DSTM32F103xB
        )

target_include_directories(${EXECUTABLE} PRIVATE
        BSP/Core/Inc
        BSP/Drivers/STM32F1xx_HAL_Driver/Inc
        BSP/Drivers/CMSIS/Device/ST/STM32F1xx/Include
        BSP/Drivers/CMSIS/Include
        )
	
target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m3
        -mthumb

        -fdata-sections
        -ffunction-sections

        -Wall

        $<$<CONFIG:Debug>:-Og>
        )

target_link_options(${EXECUTABLE} PRIVATE
        -T${CMAKE_SOURCE_DIR}/BSP/STM32F103C8Tx_FLASH.ld
        -mcpu=cortex-m3
        -mthumb
        -specs=nano.specs
        -lc
        -lm
        -lnosys
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        )
		
# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)