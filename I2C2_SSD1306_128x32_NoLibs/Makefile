
BUILD_DIR = build

TARGET = firmware

C_SOURCES =  \
	main.c 

ASM_SOURCES = startup.S

GCC_PATH = ../toolchain/RISC-V Embedded GCC/bin
OCD_PATH = ../toolchain/OpenOCD/bin

OCD = "$(OCD_PATH)/openocd"

PREFIX = riscv-none-embed-

CC = "$(GCC_PATH)/$(PREFIX)gcc"
AS = "$(GCC_PATH)/$(PREFIX)gcc" -x assembler-with-cpp
CP = "$(GCC_PATH)/$(PREFIX)objcopy"
SZ = "$(GCC_PATH)/$(PREFIX)size"

HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S
 
CPU = -march=rv32ecxw

MCU = $(CPU) -mabi=ilp32e -msmall-data-limit=0 -msave-restore -fmessage-length=0 -fsigned-char -ffunction-sections -fdata-sections -fno-common -Wunused -Wuninitialized

OPT = -Os

C_INCLUDES = -I. 

ASFLAGS = $(MCU) $(OPT)

CFLAGS += $(MCU) $(C_INCLUDES) $(OPT)

CFLAGS += -MMD -MP -MF"$(@:%.o=%.d)"

LDSCRIPT = link.ld

LDFLAGS = $(MCU) -g --specs=nano.specs -T$(LDSCRIPT) -nostartfiles -Xlinker \
	--gc-sections -Wl,-Map=$(BUILD_DIR)/$(TARGET).map --specs=nosys.specs $(OPT)
	
OPENOCD_CFG = wch-riscv.cfg


all: flash

flash: erase 
	$(OCD) -f $(OPENOCD_CFG) -c init -c halt  -c "program $(BUILD_DIR)/$(TARGET).hex" -c wlink_reset_resume -c exit
	
erase: build
	$(OCD) -f $(OPENOCD_CFG) -c init -c halt -c "flash erase_sector wch_riscv 0 last" -c exit

build: $(BUILD_DIR)/$(TARGET).elf $(BUILD_DIR)/$(TARGET).hex $(BUILD_DIR)/$(TARGET).bin


OBJECTS = $(addprefix $(BUILD_DIR)/,$(notdir $(C_SOURCES:.c=.o)))
vpath %.c $(sort $(dir $(C_SOURCES)))

OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES:.S=.o)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))

$(BUILD_DIR)/%.o: %.c Makefile | make_folder
	$(CC) -c $(CFLAGS) -g -std=gnu99 -MT"$(@)" -c "$<" -o "$@"

$(BUILD_DIR)/%.o: %.s Makefile | make_folder
	$(AS) -c $(CFLAGS) -g -MT"$(@)" -c "$<" -o "$@"

$(BUILD_DIR)/$(TARGET).elf: $(OBJECTS) Makefile | make_folder
	$(CC) $(OBJECTS) $(LDFLAGS) -o $@
	$(SZ) $@

$(BUILD_DIR)/%.hex: $(BUILD_DIR)/%.elf | make_folder
	$(HEX) $< $@
	
$(BUILD_DIR)/%.bin: $(BUILD_DIR)/%.elf | make_folder
	$(BIN) $< $@	
	

make_folder:
	@if not exist $(BUILD_DIR) (mkdir $(BUILD_DIR))	

clean:
	@-rmdir /s /q build

reset: 
	$(OCD) -f $(OPENOCD_CFG) -c init -c reset -c resume -c exit
