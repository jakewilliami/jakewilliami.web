+++
title = "Steps to create a bootable USB drive for Linux"
date = 2019-11-02
+++

These steps work near-flawlessly on a OSX operating system to create Linux bootable drives.

  1. Download the ISO (or torrent, and extract the ISO);
  2. Erase USB using Disk Utility application.  It should be `Mac OS Extended (Journaled)` and `GUID Partition Map`;
  3. Run `diskutil list` to get the number (`N`) of the disk you are putting Linux onto, and then run `diskutil unmoundDisk /dev/diskN`;
  4. `cd ~/Downloads`;
  5. `hdiutil convert -format UDRW -o outputfile.img sourcefile.iso`
  6. `sudo dd if=<file made in step 5> of=/dev/diskN bs=1m`
  7. DO NOT CLICK THE POP-UP!  Once step 6 is done, simply run `disktuil eject /dev/diskN`.  (Now you can click ignore).
