---
title: "Playing with old mac computers"
date: 2019-10-16
---

My 2008 iMac had a flashing lock symbol on it when I turned it on.

Using results from [here](https://reddit.com/r/osx/comments/a7dn34/best_osx_for_mackbook_mid_2009_8gb_ran_2_tb/ee4ak4v), [here](https://reddit.com/r/osx/comments/8dk8im/os_x_el_capitan_app/), and [here](osxdaily.com/2015/09/30/create-os-x-el-capitan-boot-install-drive/), I fixed it!  (Well, kind of.  It didn't work with El Cap., but worked with Mavericks).  After about a year or finding "solutions" online not to work, this is all it took? (In my defence, I either have a faulty option key to bring up the start menu, or a fault USB port).  (I also couldn't recall the password for a while, but I remember it now that it doesn't matter anymore).

Now that is has been so long, I wonder if I should install linux?

---

Note from the future: these [macadmin-scripts](https://github.com/munki/macadmin-scripts) are a really great tool to download old Apple Operating Systems, as Apple tries extremely hard not to let you downgrade a machine, and even makes the OS downloads quite tricky...

---

After wondering for a long time whether or not I should install Linux on this old iMac, I decided I want to...

I wrote some notes on Linux on the 17th October, 2019:
  - In aggressive to convervative order, we have Manjaro &rarr; Fedora &rarr; Ubuntu &rarr; Debian.
  - Gnome is more mac-like, Plasma is more windows-like (I think).
  - Lubuntu, Xubuntu, Mint XFCE are lighter distributions.

While I was playing with these distributions, and operating systems, and gettting a better understanding of using the command line, I wrote a very enamoured note about package managers, because I couldn't believe how easy it was to manage everything with the help of them.

I also found this helpful for making a bootable drive (but more on that later, as this is what I used to make the El Cap. bootable drives!):
```bash
sudo /Applications/install-sys.app/Contents/Resources/createinstallmedia --volume /Volumes/disk-vol --applicationpath /Ammplications/install-sys.app --nointeration
```

I have tried (and failed) using this method found [here](https://lewan.com/2012/02/10/making-a-bootable-usb-stick-on-an-apple-mac-os-x-from-an-iso).  I also tried using Etcher.io, but it didn't work.

*Note: see a later blog post from 2nd November, to see a good method of making bootable drives for Linux OS's, on a Mac.*

---

A funny turn of events: after, on 24th October, 2019, I tried installing OS X Lion on my 2007 MacBook.  It didn't have anough RAM.  It needed 2 GB...

---

After getting my 2002 iMac, I tried to get OS Tiger.  I found it [here](macintoshgarden.org/apps/mac-osx-mac-os-10-ppc) and downloaded `Tiger_4_6.dmg_.zip`.  I also found [a Snow Leopard install](archive.org/details/SnowLeopardInstall) for my 2008 iMac, which worked!

To get Ruby worjing on my old iMac, I had to run
<!-- This is actually tcsh but only sh syntax highlighting available -->
```sh
\curl -sSL https://get.rvm.io | bash -s stable && source ~/.rvm/scripts && rvm install 2.2.3 --disable-binary && brew install icu4c cmake pkg-config
```

---

I decided (based on notes from 3rd November, 2019) that I would install Linux on my 2008 iMac.  I tried fedora, and using [this](https://cyberciti.biz/faq/fedora-linux-install-broadcom-wl-sta-wireless-driver-for-bcm43228) and [this](https://forums.fedoraforum.org/showthread.php?281874-Activation-of-network-connection-failed), I managed to install the correct drivers for my `BCM4321` interface, and get WiFi working!
