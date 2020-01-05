#!Makefile
#
# --------------------------------------------------------
#
#    hurlex 这个小内核的 Makefile
#    默认使用的C语言编译器是 GCC、汇编语言编译器是 nasm
#
# --------------------------------------------------------
#

# patsubst 处理所有在 C_SOURCES 字列中的字（一列文件名），如果它的 结尾是 '.c'，就用 '.o' 把 '.c' 取代
C_SOURCES = $(shell find . -name "*.c")
C_OBJECTS = $(patsubst %.c, %.o, $(C_SOURCES))
S_SOURCES = $(shell find . -name "*.s")
S_OBJECTS = $(patsubst %.s, %.o, $(S_SOURCES))

# Mac OS 下 工具链路径
LLVM_BIN = /usr/local/Cellar/llvm/9.0.0_1/bin/
NASM_BIN = /usr/local/Cellar/nasm/2.14.02/bin/
CC = $(LLVM_BIN)/clang
LD = $(LLVM_BIN)/clang
ASM = $(NASM_BIN)/nasm

C_FLAGS = -c -Wall -m32 -ggdb -g -nostdinc -fno-builtin -fno-stack-protector -I include -target i386-none-elf
# LD_FLAGS = -T scripts/kernel.ld -m elf_i386 -nostdlib 
LD_FLAGS = -target i386-linux-elf -nostdlib -Wl,scripts/kernel.ld -fuse-ld=lld
ASM_FLAGS = -f elf -g -F stabs

all: $(S_OBJECTS) $(C_OBJECTS) link

# The automatic variable `$<' is just the first prerequisite
.c.o:
	@echo 编译代码文件 $< ...
	$(CC) $(C_FLAGS) $< -o $@

.s.o:
	@echo 编译汇编文件 $< ...
	$(ASM) $(ASM_FLAGS) $<

link:
	@echo 链接内核文件...
	$(LD) $(LD_FLAGS) $(S_OBJECTS) $(C_OBJECTS) -o hx_kernel

.PHONY:clean
clean:
	$(RM) $(S_OBJECTS) $(C_OBJECTS) hx_kernel

.PHONY:update_image
update_image:
	mkdir mntkernel
	$(eval $@_DSK := $(shell hdid -nomount ./floppy.img))
	mount_msdos $($@_DSK) ./mntkernel
	cp hx_kernel ./mntkernel/hx_kernel
	sleep 1
	umount ./mntkernel
	hdiutil detach $($@_DSK)
	rmdir mntkernel

.PHONY:mount_image
mount_image:
	mkdir mntkernel
	$(eval $@_DSK := $(shell hdid -nomount ./floppy.img))
	mount_msdos $($@_DSK) ./mntkernel

.PHONY:umount_image
umount_image:
	umount ./mntkernel
	hdiutil detach $($@_DSK)
	rmdir mntkernel

.PHONY:qemu
qemu:
	qemu-system-i386 -m 32m -boot order=a -fda floppy.img

.PHONY:bochs
bochs:
	bochs -f scripts/bochsrc.txt

.PHONY:debug
debug:
	qemu-system-i386 -S -s -boot order=a -fda floppy.img  &
	sleep 1
	cgdb -x scripts/gdbinit

