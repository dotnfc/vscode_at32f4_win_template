# vscode_at32f4_win_template
artery at32f403acgu7 vscode template for windows

* cmake_project - use artery firmware library, cmake, build and debug
* pio_project - use platformio, build and debug
  
  > you can install artery_pio from [here](https://github.com/ArteryTek/platform-arterytekat32)

NOTE 

i have not use openocd to debug target, just JLINK & STLink. so here is the toolchain for compiling and debugging:

- [STM32CubeIDE](https://www.st.com/STM32CubeIDE) - for Jlink/STLink GDB Server and CubeProgramer
   
- [STM32CubeMX](https://www.st.com/STM32CubeMX)  - to generate a Makefile project

- [cmake](https://cmake.org/download/)


just like the [vscode_arm_win_template](https://github.com/dotnfc/vscode_arm_win_template).

<hr>

*2023/10/20*

