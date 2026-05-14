---
title: Setup Ubuntu With Systemd Boot and Secure Boot
date: 2026-05-02T03:51:25-06:00
draft: false
featuredImage: ""
externalLink: ""
---

I recently switched to using systemd-boot as my bootloader for my Ubuntu installation, and I wanted to share my experience with setting it up, especially with Secure Boot enabled. This guide will walk you through the steps to get systemd-boot working on your Ubuntu system while keeping Secure Boot intact.

## Prerequisites

Before we begin, make sure you have the following:

- A UEFI-compatible system
- Ubuntu installed (preferably on a GPT partitioned disk)
- A USB drive for creating a bootable installer (if you need to reinstall Ubuntu)
- Basic knowledge of terminal commands
- Backup your important data before making any changes to your system
- A working internet connection (for downloading necessary packages)
- Prior experience with configuring UEFI settings in your BIOS/UEFI firmware

The later is the most important point, as you will need to know how to access your UEFI settings to disable/enable Secure Boot, enable installation of custom DB keys and placing UEFI system into admin/audit mode. Without this knowledge, you may find it difficult to follow the steps outlined in this guide and/or may risk bricking your system if you make a mistake in the UEFI settings. If you are unsure about how to access your UEFI settings, consult your motherboard or laptop manufacturer's documentation for instructions.

> [!WARNING]
> Modifying UEFI settings can be risky and may lead to a non-bootable system. You have been warned. Make sure to follow the instructions carefully and double-check each step before proceeding.

At this point you should have a good understanding of the risks involved and the necessary precautions to take when modifying your UEFI settings. If you are ready to proceed, let's get started with setting up systemd-boot on your Ubuntu installation while keeping Secure Boot enabled.

## Step 1: Disable Secure Boot

First thing that needs to be done is to disable Secure Boot, place UEFI system into admin/audit mode and enable installation of custom DB keys. Doing this will allow us to only have to reboot into UEFI settings once more to enable Secure Boot again after we have installed our custom DB key and signed the necessary bootloader files.

The following is a general guide on how to disable Secure Boot on the most common OEM systems (laptops and desktops).

| **Manufacturer** | **Keys**   | **Additional Notes**                                                                   |
|------------------|------------|----------------------------------------------------------------------------------------|
| Dell/Alienware   | F2 or F12  | Pressing F12 may bring up the boot menu, while F2 will take you to the UEFI settings.  |
| HP               | F10 or Esc | Pressing Esc may bring up the boot menu, while F10 will take you to the UEFI settings. |
| Lenovo           | F1 or F2   | Pressing F1 may take you to the UEFI settings, while F2 may bring up the boot menu.    |
| ASUS             | F2 or Del  | Pressing Del may take you to the UEFI settings, while F2 may bring up the boot menu.   |
| Acer             | F2 or Del  | Pressing Del may take you to the UEFI settings, while F2 may bring up the boot menu.   |
| MSI              | Del or F2  | Pressing Del may take you to the UEFI settings, while F2 may bring up the boot menu.   |
| Framework        | F2 or Del  | Pressing Del may take you to the UEFI settings, while F2 may bring up the boot menu.   |

One of the better ways to access your UEFI settings is to use `systemctl` to reboot directly into the firmware setup. This can be done with the following command:

```bash
sudo systemctl reboot --firmware-setup
```

Once you have accessed your UEFI settings, look for the Secure Boot option and disable it. The location of this option may vary depending on your motherboard or laptop manufacturer, but it is usually found under the "Boot" or "Security" tab. After disabling Secure Boot, save your changes and exit the UEFI settings.

> [!NOTE]
> On some systems, the admin/audit mode may only be enabled once. This is true of Dell's and Alienware's. After a reboot from a running system, the admin/audit mode will be disabled once again. This is to prevent unauthorized users from making changes to the UEFI settings without proper authentication.

Look for Security > Secure Boot. There should be an option to enable/disable Secure Boot. From there you may find **Admin Mode** or **Audit Mode**; enable it. Then look for an option labeled something like "Key Management" or "Key Enrollment". Setting this to "Custom" or "User" will allow you to enroll your own keys, which is necessary for signing the bootloader files later on.

## Step 2: Install SBCTL (Secure Boot Control)

[SBCTL](https://github.com/Foxboron/sbctl) is a relatively user-friendly tool for managing Secure Boot keys and signing bootloader files. It simplifies the process of enrolling custom keys and signing the necessary files to work with Secure Boot. Likewise, the same keys created with **sbctl** can be used to sign **dkms** built kernel modules in order to have them load with Secure Boot enabled.

Ad the Apt repository for sbctl:

```bash
echo "deb [signed-by=/usr/share/keyrings/home_jloeser_secureboot.gpg] https://download.opensuse.org/repositories/home:/jloeser:/secureboot/xUbuntu_24.04/ /" | sudo tee /etc/apt/sources.list.d/home_jloeser_secureboot.list
```

Then add the GPG key for the repository:

```bash
curl -fsSL https://download.opensuse.org/repositories/home:/jloeser:/secureboot/xUbuntu_24.04/Release.key | gpg --dearmor -o /usr/share/keyrings/home_jloeser_secureboot.gpg
```

Install sbctl and systemd-boot:

```bash
sudo apt update && sudo apt install --yes sbctl systemd-boot
```

### SBCTL Setup

Now that we have the repository added, we can install sbctl:

```bash
sudo sbctl setup --setup
```

This command will generate the necessary keys and enroll them into the UEFI firmware. Follow the prompts to complete the setup process. You will be asked to create a password for the keys, which will be used later when signing the bootloader files.

> [!NOTE]
>If you are dual-booting with Windows, you well need to enroll the Microsoft UEFI CA key as well. This can be done with the following command: [^UEFI CA Keys]
>
>```bash
>sudo sbctl enroll --microsoft
>```

### Verify Key Enrollment

After completing the setup process, you can verify that the keys have been enrolled correctly by running the following command:

```bash
sudo sbctl list-enrolled-keys
```

## Step 3: Configure systemd-boot

As of Ubuntu 24.04, systemd-boot generally does not require any additional configuration to work. There is no need to create custom loader entries or modify the default configuration file. However, since [Canonical](https://ubuntu.com) has not yet fully embraced systemd-boot as the default bootloader. A file needs to be created.

Now set the kernel's command line option `cat /proc/cmdline`: Copy the output and write it to `/etc/kernel/cmdline`:

```bash
echo "$(cat /proc/cmdline)" | sudo tee /etc/kernel/cmdline
```

The content should look something like this:

```ini {filename="/etc/kernel/cmdline"}
# Example content (adjust root UUID to match yours):
# root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx ro quiet splash
BOOT_IMAGE=/BOOT/ubuntu_abq4fl@/vmlinuz-6.17.0-20-generic root=ZFS=rpool/ROOT/ubuntu_abq4fl ro quiet splash vt.handoff=1
```

As you can see that I'm using ZFS as the filesystem for my root partition, so the `root` parameter is set to `ZFS=rpool/ROOT/ubuntu_abq4fl`. If you are using a different filesystem, such as ext4, the `root` parameter will be set to something like `root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`, where the UUID corresponds to your root partition.

Now regenerate boot entries for all installed kernels with the following command:

Verify that systemd-boot is installed and set up correctly by running the following command: `bootctl status` and `efibootmgr`. You should see systemd-boot listed as the default bootloader. If it is not set as the default, you can set it with the following command:

```bash
for kernel in /boot/vmlinuz-*; do
    a=$(basename "$kernel" | sed 's/vmlinuz-//')
    kernel-install add "$a" /boot/$(basename "$kernel")
done
```

Because **sbctl** was installed along with systemd-boot once the vmlinuz is regenerated **sbctl** should detect this and automatically sign the new kernel files with the keys we created. However, if for some reason this does not happen, you can manually sign the kernel files with the following command:

```bash
while read -r file; do
    sudo sbctl sign "$file"
done < <(sbctl verify | grep "is not signed")
```

This command will sign all the boot related files that are not currently signed with the keys we created. This includes grub and shim which if everything is working correctly will be removed from the system after a reboot.

Skip the next part of this step if you are not using any **dkms** modules or do not have any installed. If you do have **dkms** modules installed, you will need to sign them as well in order for them to load with Secure Boot enabled. However, it is worth at lease pointing **dkms** to sign any future built modules with the correct keys. This will ensure that you do not have to manually sign them every time a new kernel is released or a new module is built.

### Step 3.1: DKMS Modules

If you have any DKMS modules installed, you will also need to sign them in order for them to load with Secure Boot enabled. Most DKMS modules are located in the `/lib/modules/$(uname -r)/updates/dkms/` directory. You can sign them with the following command:

```bash
for module in /lib/modules/$(uname -r)/updates/dkms/*.ko.zst; do
    zstd -d "$module" -c | /usr/scr/linux-headers-$(uname -r)/scripts/sign-file sha256 \
        /var/lib/sbctl/keys/db/db.key \
        /var/lib/sbctl/keys/db/db.pem
    zstd -f $module
    rm -f $module
done
```

In order to ensure that new DKMS modules are automatically signed with the correct keys when built: Two thinks need to be done. The first is to create a `der` certificate that *kmodsign* requires. The second is to create an *dkms* `sbctl.conf` file in the `/etc/dkms/framework.conf.d/` directory with the following content:

```console
openssl x509 -in /var/lib/sbctl/keys/db/db.pem -outform DER -out /var/lib/sbctl/keys/db/db.der
```

Creating the config file.

```ini {filename="/etc/dkms/framework.conf.d/sbctl.conf"}
# Path to the generated keys from sbctl setup
mok_signing_key="/var/lib/sbctl/keys/db/db.key"
mok_certificate="/var/lib/sbctl/keys/db/db.der
"
```

## Step 4: Verify Everything is Working

To verify that everything is working correctly, you can check the following:

- Check that systemd-boot is being used as the bootloader by running the following command: `bootctl` and `efibootmgr`.

You should see something like this in the output of `efibootmgr`: Notice how **\EFI\systemd-bootx64.efi** is last in the string that is what you are looking for.

```console {filename="efibootmgr"}
BootCurrent: 0000
Timeout: 0 seconds
BootOrder: 0000
Boot0000* Linux Boot Manager	HD(1,GPT,5e9de057-55c7-4628-b155-23d95a0e2267,0x800,0x219800)/File(\EFI\systemd\systemd-bootx64.efi)
```

## Step 5: Re-enable Secure Boot

Now that we have everything set up and the necessary files signed, we can re-enable Secure Boot in the UEFI settings. Reboot your system and access the UEFI settings again. Look for the Secure Boot option and enable it. Save your changes and exit the UEFI settings. After rebooting, your system should now be using systemd-boot as the bootloader with Secure Boot enabled.

If everything is working correctly and you are dual-booting with Windows, you should see the option a systemd-boot menu. Otherwise the boot loader will be hidden.

```bash
sudo systenctl reboot --firmware-setup
```

You will not have to worry about disabling admin/audit mode, this was done automatically during **POST**. The only thing that needs to be done is to re-enable Secure Boot. Once that is done, save and exit the UEFI settings, and your system should now be using systemd-boot as the bootloader with Secure Boot enabled.

If everything goes well, you should be able to boot into your Ubuntu installation without any issues. You can verify that Secure Boot is enabled by running the following command `sbctl status`.

[^UEFI CA Keys]: Microsoft UEFI CA keys are created by Microsoft and on systems that are only going to be used for Linux can be safely removed. However, if at any point you want to dual-boot with Windows, you will need to a full BIOS/UEFI recovery to get them back.
