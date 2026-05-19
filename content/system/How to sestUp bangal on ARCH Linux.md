+++
title = "How to sestUp bangal on ARCH Linux"
date = "2026-05-19"
author = "FAKE_SYSTEM"
+++

# How to setup bangla and ibus for switching keyboard layout in arch

## Package you need

First of all you have to install ibus and two fonts family.

```bash
sudo pacman -S ibus ttf-indic-otf harfbuzz 
```

Then for avro you have install the avro with yay

```bash
yay -S ibus-avro
```

Then just run `ibus-setup` and add avro in input method and you are good to go. You can switch between keyboard layout by pressing `super-space` or you can change it from the setting.

```bash
ibus-setup
```
