# Raspberry Pi

## Preparing SD Card

* List all Devices and find SD Card `diskutil list`
* Format SD Card with FAT32 and give it a name `sudo diskutil eraseDisk FAT32 <sd_card_name> MBRFormat /dev/disk<Number>`
* list devices again and check

## Make a bootable SD Card

* unmount disk / volume `diskutil umount /dev/disk<Number>`
* copy os image to sd card `sudo dd bs=4m if=image_file_name.img of=/dev/disk<Number>`
* eject sd card `diskutil eject /dev/disk<Number>`

## Other things

* converting iso to img `hdiutil convert -format UDRW -o ~/path/to/target.img ~/path/to/source.iso`
* unzip xz files with `gunzip` or `xz` (install via brew)