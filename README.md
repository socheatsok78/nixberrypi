## Installing NixOS on a Raspberry Pi

This tutorial assumes you have a [Raspberry Pi 4 Model B with 4GB RAM](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/).

Before starting this tutorial, make sure you have all the necessary hardware:
- HDMI cable/adapter.
- 8GB+ SD card
- SD card reader (in case your machine doesn’t have an SD slot).
- Power cable for your Raspberry Pi.
- USB keyboard.

> [!NOTE]
> This tutorial was written for the Raspberry Pi 4B. Using a previously supported model like the 3B or 3B+ is possible with some modifications to this tutorial.

## Booting NixOS live image

> [!NOTE]
> Booting from USB may require an EEPROM firmware upgrade. This tutorial boots from an SD card to avoid such hiccups.

To prepare the AArch64 image on another device with Nix, run the following commands:

```sh
nix-shell -p wget zstd

[nix-shell:~]$ wget https://hydra.nixos.org/build/226381178/download/1/nixos-sd-image-23.11pre500597.0fbe93c5a7c-aarch64-linux.img.zst
[nix-shell:~]$ unzstd -d nixos-sd-image-23.11pre500597.0fbe93c5a7c-aarch64-linux.img.zst
[nix-shell:~]$ dmesg --follow
```

> [!NOTE]
> You can download a recent image from [Hydra](https://hydra.nixos.org/job/nixos/trunk-combined/nixos.sd_image.aarch64-linux), clicking on the latest successful build (marked with a green checkmark), and copying the link to the build product image.

> [!NOTE]
> It may be more convenient to use a software like [Etcher](https://www.balena.io/etcher/) to flash the image to your SD card if you are on a system where it’s available.

Your terminal should be printing kernel messages as they come in.

Plug in your SD card and your terminal should print what device it got assigned, for example `/dev/sdX`.


Press `Ctrl+C` to stop `dmesg --follow`.

Copy NixOS to your SD card by replacing `sdX` with the name of your device in the following command:

```sh
$ sudo dd if=nixos-sd-image-23.11pre500597.0fbe93c5a7c-aarch64-linux.img of=/dev/sdX bs=4096 conv=fsync status=progress
```

Once that command exits, **move the SD card into your Raspberry Pi and power it on**.

You should be greeted with a fresh shell.

In case the image doesn’t boot, it’s worth [updating the firmware](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#bootloader_update_stable) and booting the image again.

## Getting internet connection

At this point you’ll need an internet connection. If you can use an ethernet cable, plug it in and skip to the next section.

Replace `SSID` and `passphrase` with your wifi credentials and run:

```sh
nmcli device wifi connect '<SSID>' password '<passphrase>'
```

Wait for the connection to be established, after a few seconds, verify the connection by running:

```sh
host nixos.org
```

If DNS resolution succeeds, you have internet connectivity and can proceed to the next section.

## Updating firmware

To benefit from updates and bug fixes from the vendor, we’ll start by updating Raspberry Pi firmware:

```sh
$ nix-shell -p raspberrypi-eeprom
$ mount /dev/disk/by-label/FIRMWARE /mnt
$ BOOTFS=/mnt FIRMWARE_RELEASE_STATUS=stable rpi-eeprom-update -d -a
```

## Installing and configuring NixOS

Using `nixos-generate-config` will generate the required minimal configuration.

Run `nixos-rebuild switch` to apply the configuration, once it finishes, you MUST `exit` the current shell and log in again as `root` without a password if you haven't set one up in the configuration.

## Making changes

It booted, congratulations.

To make further changes to the configuration, [search through NixOS options](https://search.nixos.org/options), edit `/etc/nixos/configuration.nix`, and update your system:

```sh
$ sudo -i
$ nixos-rebuild switch
```

## Next steps

- Once you have a working OS, try upgrading it with `nixos-rebuild switch --upgrade` to install more recent package versions, and reboot to the old configuration if something broke.
- To enable hardware acceleration for a nice graphical desktop experience, add the [nixos-hardware](https://github.com/nixos/nixos-hardware) module to your configuration:
  ```nix
  imports = [
    "${fetchTarball "https://github.com/NixOS/nixos-hardware/tarball/master"}/raspberry-pi/4"
  ];
  ```
  We recommend pinning the reference to `nixos-hardware`: [Pinning Nixpkgs](https://nix.dev/reference/pinning-nixpkgs#ref-pinning-nixpkgs)
- To tweak bootloader options affecting hardware, [see `config.txt` options](https://www.raspberrypi.org/documentation/configuration/config-txt/). You can change these options by running `mount /dev/disk/by-label/FIRMWARE /mnt` and opening `/mnt/config.txt`.

## References
- [NixOS on ARM](https://nixos.wiki/wiki/NixOS_on_ARM)
- [Installing NixOS on a Raspberry Pi](https://nix.dev/tutorials/nixos/installing-nixos-on-a-raspberry-pi)
