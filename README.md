# BIOS-Bootloader-winload.sys
a bootloader for a windows compatible operating system in the bios pre-boot-environment

not much to see. i just started. is more for testing github

on a pc the operating-system is loaded by a bootloader. we have now 2 different pre-boot-environments: UEFI or BIOS.
UEFI is the newer variant. BIOS can only boot from a disk smaller then 2TB.
A newer PC with UEFI can also boot in BIOS-mode in CSM.
A UEFI-PC can only boot a 64bit-operating-system.

BIOS-preboot-environment:
first the bios initialize the hardware and give the user a choice to enter the bios configuration (F2 or other keys).
then bios looks at the selected boot-disk and load and start its MBR (master boot record) boot code.
this MBR is always in the firstsector (512 bytes) of the disk.

if you buy a new disk (ssd or hdd or usb stick) you must intialize it first. 
on windows 10: diskpart, list disk, select disk X (x the index of new disk: ie "2"), clean (warning all data on that disk will be erased). this will write a mbr (master boot resord) in the first sector (512bytes) of the disk. the mbr contains a mbr-bootcode and a partition table with 4 entries. the 4 entries will all be emtpy (all bytes null). 

now create a primary partion. you can use all avail sectors or use less. i use 80MB.
now format the partion/volume with Fat32.
now set the partion as active (bootable): in diskpart: select disk X, select part 1. active.

now the bios can start with this disk. the mbr bootcode will look at the partion table inside the mbr for a active partition and load the first sector of that partition (vbr=volume boot record) and start the bootcode there. because we formated with fat32 there will be the ms bootcode for fat/windows10. it will look for a bootmgr file in the partition and show a error message "no operating-system found".

we need to patch this bootcode in the first sector of the partion (often in sector 0x800) with our bootcode.
our bootcode is 1024 bytes long in fat32.bin.  use bootsect.exe e: to patch the volume e: 

our bootcode will look for a file "winload.sys" in the volume. if the file is not found the bootcode will print a error message "winload.sys not found". if the file is there we load it and start it.

winload use BIOS services to load files and to print messages on the screen.

winload.sys load the operating-system:
first winload.sys will examine the memory and the ACPI data from the bios.
on the volume should be the directory "Amparo" with system32 subdirectory.
from Amparo/system32/config it will load the file "system" as the system-hive (registry).
in that system-hive is the configuration on how to load the operating-system.

winload will now load the kernel module 'ntoskrnl.exe', the hal 'hal.dll', 'bootvid.dll' and 'kdXXX.dll'.
winload will now load the boot drivers.
winload prepare a LOADER_PARAMETER_BLOCK with all the data about all this.
now winload will switch the processor to 32bit protected mode and prepare the virtual adresses for the memory.

now winload will activate the kernel. jumt to kernel: KiSystemStartup(LOADER_PARAMETER_BLOCK *loaderData).
now the kernel will initialize the operating-system.

How to build winload.sys
we use visual studio. winload.sys is a native 
