+++
author = "Artem Derkach"
title = "Embedded Development"
date = "2021-11-30"
description = "Notes for setting embedded development workflow"
tags = [
    "embedded",
]
draft = true
+++

Notes for setting embedded development workflow. 
<!--more-->

Notes below describing flow for `STM32F446RE` which is `STM32F` family. 
Nevertheless, this notes can be used for other MCUs.
<br><br>


## Content
software components
- [Linker Script]({{< ref "#linker-script" >}})
- [Startup File]({{< ref "#startup-file" >}})
- [Libraries]({{< ref "#libraries" >}})
- [Usage]({{< ref "#usage" >}})
  
software development components
- [Documentation]({{< ref "#documentation" >}})
- software development environments
- build tools
- [Build Process]({{< ref "#build-process" >}})
- flash
- debug
- microcontroller documentation
<br><br>

## Linker Script
[article about linker script](https://blog.thea.codes/the-most-thoroughly-commented-linker-script/)
Linker script combines `.o` into single `.elf`.<br>
Main things to keep in mind is 2 parts form linker file:
```
MEMORY {
	FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
	SRAM (rwx) : ORIGIN = 0x20000000, LENGTH = 128K
}

SECTIONS {
	.isr_vector : {
		*(.isr_vector)
	} > FLASH

	.text : {
	    *(.text)
	} > FLASH
}
```
Not to be confused, `LENGTH` is defined in KibiBytes
<br><br>

### Common Options
- `-T<file>` add a `<file>` to linker process
- `-nostartfiles` do not use the standard system startup files when linking
- 

## Startup File
To better understand what is happening in startup file, we need to keep in mind what is happening before it.
Here is few lines from about reset sequence:
> After reset and before the processor starts executing the program, the Cortex-M
processors read the first two words from the memory. The beginning of
the memory space contains the vector table, and the first two words in the vector table are the
initial value for the Main Stack Pointer (MSP), and the reset vector, which
is the starting address of the reset handler. After these two words are read by the processor, the processor then
sets up the MSP and the Program Counter (PC) with these values.
<br><br>

## Libraries
Libraries or Packs is a collection of software to help you with development:
- CMSIS modules
- Hardware Access Layer (HAL) + Low Level (LL)
- Drivers
- Examples

Packs for `STM32F`:
- [STM32CubeF4](https://github.com/STMicroelectronics/STM32CubeF4)
- [Keil Packs](https://www.keil.com/dd2/pack/) includes all available packs and can be used for different controllers
<br><br>

## Usage
Minimal steps to use peripheral:
- enable clock
- set peripheral to be defined as one of function (input, output ...)
<br><br>

## Documentation
Board should have 4 types of document:
- datasheet
- reference manual
- user manual
- programming manual
Can be found on official site [here](https://www.st.com/en/microcontrollers-microprocessors/stm32f446re.html).<br>
Also [this site](https://doc.riot-os.org/group__boards__nucleo-f446re.html) go everything collected together.
<br><br>


## Build Process
As build tool `arm-none-eabi-gcc` is used, it is a `gcc` variation for `ARM`.
[gcc docs](https://gcc.gnu.org/onlinedocs/gcc-11.2.0/gcc/)

Build process consists of 4 steps:
- preprocess (`-E` to stop after this stage). 
- assemble (`-S` to stop after this stage)
- compile (`-c` to stop after this stage)
- link
### Compiler Flags
### Machine-Dependent Options
Option starts with `-m`

[Available arm options](https://gcc.gnu.org/onlinedocs/gcc-11.2.0/gcc/ARM-Options.html)

Cortex-M options ([source](https://launchpadlibrarian.net/177524521/readme.txt)):
```
--------------------------------------------------------------------
| ARM Core | Command Line Options                       | multilib |
|----------|--------------------------------------------|----------|
|Cortex-M0+| -mthumb -mcpu=cortex-m0plus                | armv6-m  |
|Cortex-M0 | -mthumb -mcpu=cortex-m0                    |          |
|Cortex-M1 | -mthumb -mcpu=cortex-m1                    |          |
|          |--------------------------------------------|          |
|          | -mthumb -march=armv6-m                     |          |
|----------|--------------------------------------------|----------|
|Cortex-M3 | -mthumb -mcpu=cortex-m3                    | armv7-m  |
|          |--------------------------------------------|          |
|          | -mthumb -march=armv7-m                     |          |
|----------|--------------------------------------------|----------|
|Cortex-M4 | -mthumb -mcpu=cortex-m4                    | armv7e-m |
|(No FP)   |--------------------------------------------|          |
|          | -mthumb -march=armv7e-m                    |          |
|----------|--------------------------------------------|----------|
|Cortex-M4 | -mthumb -mcpu=cortex-m4 -mfloat-abi=softfp | armv7e-m |
|(Soft FP) | -mfpu=fpv4-sp-d16                          | /softfp  |
|          |--------------------------------------------|          |
|          | -mthumb -march=armv7e-m -mfloat-abi=softfp |          |
|          | -mfpu=fpv4-sp-d16                          |          |
|----------|--------------------------------------------|----------|
|Cortex-M4 | -mthumb -mcpu=cortex-m4 -mfloat-abi=hard   | armv7e-m |
|(Hard FP) | -mfpu=fpv4-sp-d16                          | /fpu     |
|          |--------------------------------------------|          |
|          | -mthumb -march=armv7e-m -mfloat-abi=hard   |          |
|          | -mfpu=fpv4-sp-d16                          |          |
|----------|--------------------------------------------|----------|
|Cortex-R4 | [-mthumb] -march=armv7-r                   | armv7-ar |
|Cortex-R5 |                                            | /thumb   |
|Cortex-R7 |                                            |          |
|(No FP)   |                                            |          |
|----------|--------------------------------------------|----------|
|Cortex-R4 | [-mthumb] -march=armv7-r -mfloat-abi=softfp| armv7-ar |
|Cortex-R5 | -mfpu=vfpv3-d16                            | /thumb   |
|Cortex-R7 |                                            | /softfp  |
|(Soft FP) |                                            |          |
|----------|--------------------------------------------|----------|
|Cortex-R4 | [-mthumb] -march=armv7-r -mfloat-abi=hard  | armv7-ar |
|Cortex-R5 | -mfpu=vfpv3-d16                            | /thumb   |
|Cortex-R7 |                                            | /fpu     |
|(Hard FP) |                                            |          |
|----------|--------------------------------------------|----------|
|Cortex-A* | [-mthumb] -march=armv7-a                   | armv7-ar |
|(No FP)   |                                            | /thumb   |
|----------|--------------------------------------------|----------|
|Cortex-A* | [-mthumb] -march=armv7-a -mfloat-abi=softfp| armv7-ar |
|(Soft FP) | -mfpu=vfpv3-d16                            | /thumb   |
|          |                                            | /softfp  |
|----------|--------------------------------------------|----------|
|Cortex-A* | [-mthumb] -march=armv7-a -mfloat-abi=hard  | armv7-ar |
|(Hard FP) | -mfpu=vfpv3-d16                            | /thumb   |
|          |                                            | /fpu     |
--------------------------------------------------------------------
```

Based on table above options for `STM32F446RE` is:
- `-mlittle-endian` (default for arm processors)
- `-mthumb` (instruction set). Can also be `-marm` or `-mthumb-interwork`
- `-mcpu=cortex-m4`
- `-march=armv7e-m`
- `-mfloat-abi=hard` (chip supports floating point instructions)
- `-mfpu=fpv4-sp-d16`

#### Architecture
`-mcpu=cortex-m4`

<br><br>


### Linker Flags (`LDFLAGS`)
`-T` - to include linker script (`-Tstm324f.ld`)
<br><br>

