---
layout: post
title: Resetting old mac computer passwords
---

We are going to cover two methods to do this in this little section.

The first method works for > 10.5 version.  It works by resetting keychain.  Reboot and press <kbd>&#8984;</kbd> + <kbd>S</kbd>.  Now in terminal, type
```tcsh
mount -uw /
launchctl load /System/Library/LaunchDaemons/com/apple/DirectoryServices.plist
ls /Users
dscl . -passwd /Users/<user> <new password>
reboot
```

Now your new password will work for the user you need to hack into!

The second method works for &leq; 10.5.  It will create a new user without root permissions, but will leave the old user there.  It does this by tricking the system into thinking it is new again.  You need to reboot and press <kbd>&#8984;</kbd> + <kbd>R</kbd>.  Now type into terminal
```tcsh
mount -uw /
rm /var/db/.AppleSetupDone
shutdown -h now
```

