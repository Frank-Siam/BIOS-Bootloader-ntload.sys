# BIOS-Bootloader-ntload.sys
a bootloader for a windows compatible operating system for the bios pre-boot-environment

not much to see. i just started.

the operating-system is loaded by a bootloader. there can be 2 different pre-boot-environments: UEFI or BIOS.
UEFI is the newer variant. BIOS can only boot from a disk smaller then 2TB.
A newer PC with UEFI can also boot in BIOS-mode in CSM.

BIOS-preboot-environment:
first the bios initialize the hardware and give the user a choice to enter the bios configuration (F2 or other keys).
then bios load and start a Boot-Loader from the boot device which was chosen in the Bios-configuration.
If he configured boot device is a hard disc, then Bios load the MBR (master boot record) boot code.
this MBR is always in the first sector (512 bytes) of the disk.

before using a new disk (ssd or hdd or usb stick) you must intialize it first. 
on windows 10: diskpart, list disk, select disk X (x the index of new disk: ie "2"), clean (warning all data on that disk will be erased). this will write a mbr (master boot resord) in the first sector (512bytes) of the disk. the mbr contains a mbr-bootcode and a partition table with 4 entries. the 4 entries will all be emtpy (all bytes null). 

now create a primary partion. you can use all avail sectors or use less. i use 80MB.
now format the partion/volume with Fat32.
now set the partion as active (bootable): in diskpart: select disk X, select part 1. active.

now the bios can start with this disk. the mbr bootcode will look at the partion table inside the mbr for a active partition and load the first sector of that partition (vbr=volume boot record) and start the bootcode there. because we formated with fat32 there will be the ms bootcode for fat/windows10. it will look for a bootmgr file in the partition and show a error message "no operating-system found".

we need to patch this bootcode in the first sector of the partion (often in sector 0x800) with our bootcode.
our bootcode is 1024 bytes long in fat32.bin.  use bootsect.exe e: to patch the volume e: 

our bootcode will look for a file "ntload.sys" in the volume. if the file is not found the bootcode will print a error message "ntload.sys not found". if the file is there we load it and start it.

ntload use BIOS services to load files and to print messages on the screen.

ntload.sys load the operating-system:
first ntload.sys will print its name and version on the screen: ntload.sys 2022-nov-1 22k
print the name of the boot device and boot partition: boot from Samsung evo600 120GB MBR-1B 60MB Fat32
on the volume should be the directory "Amparo" with system32 subdirectory.
from Amparo/system32/config it will load the file "system" as the system-hive (registry).
in that system-hive is the configuration on how to load the operating-system.
print on screen: system-hive 64k ok 

ntload will now load the kernel module 'ntoskrnl.exe', the hal 'hal.dll', 'bootvid.dll' and 'kdcom.dll'.
ntload will now load the boot drivers.
ntload prepare a LOADER_PARAMETER_BLOCK with all the data about all this.
now ntload will switch the processor to 32 or 64bit protected mode and prepare the virtual adresses for the memory.

now ntload will activate the kernel. jumt to kernel: KiSystemStartup(LOADER_PARAMETER_BLOCK *loaderData).
now the kernel will initialize the operating-system.

How to build ntload.sys
we use visual studio. ntload.sys is a native 
