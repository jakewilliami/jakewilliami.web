+++
title = "Increasing trackpad speed on a Mac more than the maximum allowed"
date = 2019-11-06
+++

You can increase trackspeed in System Preferences, but that only goes up so far.

To see the current speed, you run
```bash
defaults read -g com.apple.trackpad.scaling
```

Initially, this outputted in `0.6875`.

To change it, you need to run
```bash
defaults write -g com.apple.trackpad.scaling <number>
```

Then
```bash
sudo reboot
```

Now your trackpad will be faster!  I set mine to `4`.
