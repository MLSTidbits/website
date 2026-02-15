---
title: How to Flash the LibreDrive Firmware on Linux
date: 2026-02-07T06:08:51-07:00
authors:
  - name: Michael Schaecher
    image: /authors/Michael.jpg
tags:
  - MakeMKV
  - Linux
  - How To
excludeSearch: false
draft: false
comments: true
---

Flashing the LibreDrive firmware may seem daunting at first even for experienced user, weather you are using Windows, Linux or MacOS. This guide will walk you through the process of flashing the LibreDrive firmware on a Linux system. Thankfully the process is quite straightforward and can be completed in just a few steps.

Before you begin the flashing process, it is important to verify that your optical drive is compatible with the LibreDrive firmware. You can check the list of compatible drives on the official LibreDrive website.[^1] If your drive is not listed, it may not be compatible with the LibreDrive firmware and you may want to consider purchasing a compatible drive before proceeding.

## Step 1: Download the MakeMKV Version 1.17.7

As of this writing, the latest version of MakeMKV is 1.18.3, but this version is not compatible with Linux or MacOS when it comes to flashing the LibreDrive firmware. Therefore, you will need to download version 1.17.7 of MakeMKV, which is version that is compatible with Linux and MacOS for flashing the LibreDrive firmware. You can download it using the following command in your terminal:

```bash
wget https://www.makemkv.com/download/old/makemkv-1.17.7.tar.gz
```

## Step 2: Extract the MakeMKV Archive

Once you have downloaded the MakeMKV archive, you will need to extract it to a directory on your system for later use.

```bash
tar -xzf makemkv-1.17.7.tar.gz
```

I recommend keeping the extracted onto a flash drive for later use, as you will need to access the MakeMKV executable from this directory in the next steps.

## Step 3: Download SDF.bin

Next, you will need to download the SDF.bin. This contains the necessary tools for `makemkvcon` to flash the LibreDrive firmware.

```bash
wget -c https://makemkv.com/sdf.bin
```

Keep the `sdf.bin` file in the same directory as the MakeMKV executable for later use.

## Step 4: Download and Flash the LibreDrive Firmware

Now that you have the necessary files, you can proceed to download and flash the LibreDrive firmware. You can do this by running the following command in your terminal:

```bash
wget -c https://download2273.mediafire.com/6ykduskt2mxgE0pFTQQ-4U598v1jadUhdvPFroqToaDinsGAEgH0QMtaPbwfZM0KnCHoMb5qJS15adSs82iOiSQSuEUD8lC3m0Ux8zEFIksakocjVumCD4JUWVOA601OT_diwWgBl5SPIucOudTIq3gb0KObWKxYyOUIDHuc_yH8/bpl3pz2brp9lquk/All+You+Need+Firmware+Pack+%28MartyMcNuts%29.zip

unzip 'All You Need Firmware Pack (MartyMcNuts).zip'
```

Navigate to the directory where the MakeMKV executable is located and run the following command to flash the LibreDrive firmware:

```bash
cd /path/to/extracted/directory
makemkv/bin/amd64/makemkvcon f \
 -d '/dev/sr0' -f sdf.bin rawflash enc \
 -i 'Firmware/[Internal|External] Drives/[DE-Make_Model-Version].bin'
```

> **NOTE**: If you need to find out what the model of your drive is, you can run the following command `sudo lshw -class disk` and look for the `product` field under the drive you want to flash. The model of your drive will be in the format of `DE-* Model Number`.

## Conclusion

Flashing the LibreDrive firmware on Linux is a straightforward process that can be completed in just a few steps. By following this guide, you should be able to successfully flash the LibreDrive firmware and enjoy the benefits of this powerful tool for your optical drive. If you encounter any issues during the flashing process, make sure to double-check that you have downloaded the correct version of MakeMKV and that you have the necessary files in the correct locations.

[^1]: LG, Buffalo, Asus and older Pioneer drives are compatible with the LibreDrive firmware [source](https://forum.makemkv.com/forum/viewtopic.php?p=79712#p79712).
