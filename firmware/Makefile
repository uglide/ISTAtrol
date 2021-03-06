###############################################################################
# Makefile for ISTAtrol heating valve controller.
#
# Copyright (C) 2016 Markus "Traumflug" Hitter <mah@jump-ing.de>
#
# This program is free software: you can redistribute it and/or modify it
# under the terms of the GNU General Public License as published by the Free
# Software Foundation, either version 3 of the License, or (at your option)
# any later version.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
# FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for
# more details.
#
# You should have received a copy of the GNU General Public License along with
# this program. If not, see <http://www.gnu.org/licenses/>.
###############################################################################

## General Flags
PROJECT = firmware

MCU = attiny2313

F_CPU = 12800000

BUILDDIR = build

AVRDUDE = avrdude
AVRDUDEFLAGS = -c stk500v2 -p $(MCU) -P /dev/ttyACM0
AVRDUDEFLAGSFAST = $(AVRDUDEFLAGS) -B 1
AVRDUDEFLAGSSLOW = $(AVRDUDEFLAGS) -B 5

TARGET = $(PROJECT).hex
CC = avr-gcc

## Options common to compile, link and assembly rules.
COMMON = -mmcu=$(MCU) -save-temps=obj

## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -DF_CPU=$(F_CPU)
CFLAGS += -Wall
CFLAGS += -Wstrict-prototypes
CFLAGS += -Winline
CFLAGS += -std=gnu99
CFLAGS += -Os
CFLAGS += -funsigned-char
CFLAGS += -funsigned-bitfields
CFLAGS += -fpack-struct
CFLAGS += -fshort-enums
CFLAGS += -fno-move-loop-invariants
CFLAGS += -fno-tree-scev-cprop
CFLAGS += -ffunction-sections
CFLAGS += -finline-functions-called-once
CFLAGS += -fverbose-asm
CFLAGS += -Wa,-adhlns=$(@:.o=.al)

## Assembly specific flags.
ASMFLAGS = $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags.
LDFLAGS = $(COMMON)
LDFLAGS += -Wl,--as-needed
LDFLAGS += -Wl,--gc-sections
LDFLAGS += -Wl,-Map,$(@:.elf=.map)

## Intel Hex file production flags.
HEX_FLASH_FLAGS = -R .eeprom -R .fuse -R .lock -R .signature

HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings

## Include Directories.
INCLUDES = -I. -I"usbdrv" -I"libs-device"

## Objects that must be built in order to link.
OBJECTS = main.o usbdrv.o usbdrvasm.o
BUILDOBJECTS = $(addprefix $(BUILDDIR)/,$(OBJECTS))

## Objects explicitly added by the user.
LINKONLYOBJECTS =


## Build
all: $(BUILDDIR) $(TARGET) $(BUILDDIR)/$(PROJECT).elf \
     $(BUILDDIR)/$(PROJECT).eep $(BUILDDIR)/$(PROJECT).lss size

## Program
program: $(PROJECT).hex
	$(AVRDUDE) $(AVRDUDEFLAGSFAST) -U flash:w:$^

## Compile
.SUFFIXES:
.SUFFIXES: .c .S .o
$(shell mkdir -p $(BUILDDIR))

$(BUILDDIR)/*.o: Makefile

$(BUILDDIR)/main.o: main.c usbdrv/usbdrv.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $< -o $@

$(BUILDDIR)/usbdrvasm.o: usbdrv/usbdrvasm.S usbdrv/usbdrv.h usbconfig.h
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $< -o $@

$(BUILDDIR)/usbdrv.o: usbdrv/usbdrv.c usbdrv/usbdrv.h usbconfig.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $< -o $@

## Link
.SUFFIXES: .elf .eep .lss .hex
$(BUILDDIR)/%.elf: $(BUILDOBJECTS)
	$(CC) $(LDFLAGS) $(BUILDOBJECTS) $(LINKONLYOBJECTS) $(LIBDIRS) $(LIBS) -o $@

%.hex: $(BUILDDIR)/%.elf
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS) $< $@

$(BUILDDIR)/%.eep: $(BUILDDIR)/%.elf
	-avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

$(BUILDDIR)/%.lss: $(BUILDDIR)/%.elf
	avr-objdump -h -S $< > $@

size: $(BUILDDIR)/$(PROJECT).elf
	@echo
	@avr-size -C --mcu=$(MCU) $(BUILDDIR)/$(PROJECT).elf | grep "Device:"
	@avr-size -C --mcu=$(MCU) $(BUILDDIR)/$(PROJECT).elf | grep "Program:"
	@avr-size -C --mcu=$(MCU) $(BUILDDIR)/$(PROJECT).elf | grep "Data:"

## Fuses
.PHONY: fuses
fuses:
	avrdude -c avrispv2 -p ${MCU} -P /dev/ttyACM0 -B 10 \
    -U lfuse:w:0xE4:m -U hfuse:w:0xDB:m -U efuse:w:0xFF:m

## Clean target.
.PHONY: clean
clean:
	-rm -rf $(BUILDDIR) $(TARGET)
