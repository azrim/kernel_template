AnyKernel3 - Flashable Zip Template for Kernel Releases with Ramdisk Modifications
by osm0sis @ xda-developers
"AnyKernel is a template for an update.zip that can apply any kernel to any ROM, regardless of ramdisk." - Koush

AnyKernel2 pushed the format further by allowing kernel developers to modify the underlying ramdisk for kernel feature support easily using a number of included command methods along with properties and variables to customize the installation experience to their kernel. AnyKernel3 adds the power of topjohnwu's magiskboot for wider format support by default, and to automatically detect and retain Magisk root by patching the new Image.*-dtb as Magisk would.

A script based on Galaxy Nexus (tuna) is included for reference. Everything to edit is self-contained in anykernel.sh.

// Properties / Variables
kernel.string=KernelName by YourName @ xda-developers
do.devicecheck=1
do.modules=1
do.systemless=1
do.cleanup=1
do.cleanuponabort=0
device.name1=maguro
device.name2=toro
device.name3=toroplus
device.name4=tuna
supported.versions=6.0 - 7.1.2
supported.patchlevels=2019-07 -

block=/dev/block/platform/omap/omap_hsmmc.0/by-name/boot;
is_slot_device=0;
ramdisk_compression=auto;
do.devicecheck=1 specified requires at least device.name1 to be present. This should match ro.product.device, ro.build.product, ro.product.vendor.device or ro.vendor.product.device from the build.prop files for your device. There is support for as many device.name# properties as needed. You may remove any empty ones that aren't being used.

do.modules=1 will push the .ko contents of the modules directory to the same location relative to root (/) and apply correct permissions. On A/B devices this can only be done to the active slot.

do.systemless=1 (with do.modules=1) will instead push the full contents of the modules directory to create a simple "ak3-helper" Magisk module, allowing developers to effectively replace system files, including .ko files. If the current kernel is changed then the kernel helper module automatically removes itself to prevent conflicts.

do.cleanup=0 will keep the zip from removing its working directory in /tmp/anykernel (by default) - this can be useful if trying to debug in adb shell whether the patches worked correctly.

do.cleanuponabort=0 will keep the zip from removing its working directory in /tmp/anykernel (by default) in case of installation abort.

supported.versions= will match against ro.build.version.release from the current ROM's build.prop. It can be set to a list or range. As a list of one or more entries, e.g. 7.1.2 or 8.1.0, 9 it will look for exact matches, as a range, e.g. 7.1.2 - 9 it will check to make sure the current version falls within those limits. Whitespace optional, and supplied version values should be in the same number format they are in the build.prop value for that Android version.

supported.patchlevels= will match against ro.build.version.security_patch from the current ROM's build.prop. It can be set as a closed or open-ended range of dates in the format YYYY-MM, whitespace optional, e.g. 2019-04 - 2019-06, 2019-04 - or - 2019-06 where the last two examples show setting a minimum and maximum, respectively.

block=auto instead of a direct block filepath enables detection of the device boot partition for use with broad, device non-specific zips. Also accepts specifically boot or recovery.

is_slot_device=1 enables detection of the suffix for the active boot partition on slot-based devices and will add this to the end of the supplied block= path. Also accepts auto for use with broad, device non-specific zips.

ramdisk_compression=auto allows automatically repacking the ramdisk with the format detected during unpack. Changing auto to gz, lzo, lzma, xz, bz2, lz4, or lz4-l (for lz4 legacy) instead forces the repack as that format, and using cpio or none will (attempt to) force the repack as uncompressed.

customdd="<arguments>" may be added to allow specifying additional dd parameters for devices that need to hack their kernel directly into a large partition like mmcblk0, or force use of dd for flashing.

slot_select=active|inactive may be added to allow specifying the target slot. If omitted the default remains active.

// Command Methods
ui_print "<text>" [...]
abort ["<text>" [...]]
contains <string> <substring>
file_getprop <file> <property>

set_perm <owner> <group> <mode> <file> [<file2> ...]
set_perm_recursive <owner> <group> <dir_mode> <file_mode> <dir> [<dir2> ...]

dump_boot
split_boot
unpack_ramdisk

backup_file <file>
restore_file <file>
replace_string <file> <if search string> <original string> <replacement string> <scope>
replace_section <file> <begin search string> <end search string> <replacement string>
remove_section <file> <begin search string> <end search string>
insert_line <file> <if search string> <before|after> <line match string> <inserted line>
replace_line <file> <line replace string> <replacement line>
remove_line <file> <line match string>
prepend_file <file> <if search string> <patch file>
insert_file <file> <if search string> <before|after> <line match string> <patch file>
append_file <file> <if search string> <patch file>
replace_file <file> <permissions> <patch file>
patch_fstab <fstab file> <mount match name> <fs match type> block|mount|fstype|options|flags <original string> <replacement string>
patch_cmdline <cmdline entry name> <replacement string>
patch_prop <prop file> <prop name> <new prop value>
patch_ueventd <ueventd file> <device node> <permissions> <chown> <chgrp>

repack_ramdisk
flash_boot
flash_dtbo
write_boot

reset_ak [keep]
setup_ak
"if search string" is the string it looks for to decide whether it needs to add the tweak or not, so generally something to indicate the tweak already exists. "cmdline entry name" behaves somewhat like this as a match check for the name of the cmdline entry to be changed/added by the patch_cmdline function, followed by the full entry to replace it. "prop name" also serves as a match check in patch_prop for a property in the given prop file, but is only the prop name as the prop value is specified separately.

Similarly, "line match string" and "line replace string" are the search strings that locate where the modification needs to be made for those commands, "begin search string" and "end search string" are both required to select the first and last lines of the script block to be replaced for replace_section, and "mount match name" and "fs match type" are both required to narrow the patch_fstab command down to the correct entry.

"scope" may be specified as "global" to force all instances of the string targeted by replace_string to be replaced. Omitted or set to anything else and it will perform the default first-match replacement.

"before|after" requires you simply specify "before" or "after" for the placement of the inserted line, in relation to "line match string".

"block|mount|fstype|options|flags" requires you specify which part (listed in order) of the fstab entry you want to check and alter.

dump_boot and write_boot are the default method of unpacking/repacking, but for more granular control, or omitting ramdisk changes entirely ("OG AK" mode), these can be separated into split_boot; unpack_ramdisk and repack_ramdisk; flash_boot respectively. flash_dtbo can be used to flash a dtbo image. It is automatically included in write_boot but can be called separately if using "OG AK" mode or creating a dtbo only zip.

Multi-partition zips can be created by removing the ramdisk and patch folders from the zip and including instead "-files" folders named for the partition (without slot suffix), e.g. boot-files + recovery-files, or kernel-files + ramdisk-files (on some Treble devices). These then contain Image.gz, and ramdisk, patch, etc. subfolders for each partition. To setup for the next partition, simply set block= (without slot suffix) and ramdisk_compression= for the new target partition and use the reset_ak command.

Similarly, multi-slot zips can be created with the normal zip layout for the active (current) slot, then resetting for the inactive slot by setting block= (without slot suffix) again, slot_select=inactive and ramdisk_compression= for the target slot and using the reset_ak keep command, which will retain the patch and any added ramdisk files for the next slot.

backup_file may be used for testing to ensure ramdisk changes are made correctly, transparency for the end-user, or in a ramdisk-only "mod" zip. In the latter case restore_file could also be used to create a "restore" zip to undo the changes, but should be used with caution since the underlying patched files could be changed with ROM/kernel updates.

You may also use ui_print "<text>" to write messages back to the recovery during the modification process, abort "<text>" to abort with optional message, and file_getprop "<file>" "<property>" and contains "<string>" "<substring>" to simplify string testing logic you might want in your script.

// Binary Inclusion
The AK3 repo includes current ARM builds of magiskboot, magiskpolicy and busybox by default to keep the basic package small. Builds for other architectures and optional binaries (see below) are available from the latest Magisk zip, or my latest AIK-mobile and FlashIt packages, respectively, here:

https://forum.xda-developers.com/showthread.php?t=2073775 (Android Image Kitchen thread)
https://forum.xda-developers.com/showthread.php?t=2239421 (Odds and Ends thread)

Optional supported binaries which may be placed in /tools to enable built-in expanded functionality are as follows:

mkbootfs - for broken recoveries, or, booted flash support for a script/app via bind mount to /tmp (deprecated/use with caution)
flash_erase, nanddump, nandwrite - MTD block device support for devices where the dd command is not sufficient
dumpimage, mkimage - DENX U-Boot uImage format support
mboot - Intel OSIP Android image format support
unpackelf, mkbootimg - Sony ELF kernel.elf format support, repacking as AOSP standard boot.img for unlocked bootloaders
elftool (with unpackelf) - Sony ELF kernel.elf format support, repacking as ELF for older Sony devices
mkmtkhdr (with unpackelf) - MTK device boot image section headers support for Sony devices
futility + chromeos test keys directory - Google ChromeOS signature support
BootSignature_Android.jar + avb keys directory - Google Android Verified Boot (AVB) signature support
rkcrc - Rockchip KRNL ramdisk image support
Optionally moving ARM builds to tools/arm and putting x86 builds in tools/x86 will enable architecture detection for use with broad, device non-specific zips.

// Instructions
Place final kernel build product, e.g. Image.gz-dtb or zImage to name a couple, in the zip root (any separate dt, dtb or recovery_dtbo, and/or dtbo should also go here for devices that require custom ones, each will fallback to the original if not included)

Place any required ramdisk files in /ramdisk and module files in /modules (with the full path like /modules/system/lib/modules)

Place any required patch files (generally partial files which go with AK3 file editing commands) in /patch

Modify the anykernel.sh to add your kernel's name, boot partition location, permissions for any added ramdisk files, and use methods for any required ramdisk modifications (optionally, also place banner and/or version files in the root to have these displayed during flash)

zip -r9 UPDATE-AnyKernel3.zip * -x .git README.md *placeholder

The LICENSE file must remain in the final zip to comply with licenses for binary redistribution and the license of the AK3 scripts.

If supporting a recovery that forces zip signature verification (like Cyanogen Recovery) then you will need to also sign your zip using the method I describe here:

http://forum.xda-developers.com/android/software-hacking/dev-complete-shell-script-flashable-zip-t2934449

Not required, but any tweaks you can't hardcode into the source (best practice) should be added with an additional init.tweaks.rc or bootscript.sh to minimize the necessary ramdisk changes. On newer devices Magisk allows these within /overlay.d - see examples.

It is also extremely important to note that for the broadest AK3 compatibility it is always better to modify a ramdisk file rather than replace it.

If running into trouble when flashing an AK3 zip, the suffix -debugging may be added to the zip's filename to enable creation of a debug .tgz of /tmp for later examination while booted or on desktop.

// Staying Up-To-Date
Now that you've got a ready zip for your device, you might be wondering how to keep it up-to-date with the latest AnyKernel commits. AnyKernel2 and AnyKernel3 have been painstakingly developed to allow you to just drop in the latest update-binary and tools directory and have everything "just work" for beginners not overly git or script savvy, but the best practice way is as follows:

Fork my AnyKernel3 repo on GitHub

git clone https://github.com/<yourname>/AnyKernel3

git remote add upstream https://github.com/osm0sis/AnyKernel3

git checkout -b <devicename>

Set it up like your zip (i.e. remove any folders you don't use like ramdisk or patch, delete README.md, and add your anykernel.sh and optionally your Image.*-dtb if you want it up there) then commit all those changes

git push --set-upstream origin <devicename>

git checkout master then repeat steps 4-6 for any other devices you support

Then you should be able to git pull upstream master from your master branch and either merge or cherry-pick the new AK3 commits into your device branches as needed.

For further support and usage examples please see the AnyKernel3 XDA thread: https://forum.xda-developers.com/showthread.php?t=2670512

Have fun!
