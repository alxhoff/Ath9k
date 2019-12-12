# Ath9k CSI Driver for Qualcomm Atheros AR93XX chips

## Credits

This work is based off of the work by [WANDS](https://wands.sg/research/wifi/AtherosCSI/). Cheers!

## About

This is modified version of the mainline Linux kernel v5.4 Ath9k Qualcomm wireless driver. Incorporating changes similar to those found [here](https://github.com/xieyaxiongfly/Atheros-CSI-Tool), only for a more relevant kernel version.

## Building

I recommend not using an old distribution such as Ubuntu 14 etc as Ubuntu itself struggles to keep up to date, but an outdated Ubuntu version are hopelessly behind and will probably have problems with a modern kernel on ancient `gcc` versions. eg. Ubuntu 14 uses `gcc` 4.8 which will give a lot of errors with a 5.4 kernel.

### Prerequisites

Standard Linux kernel prerequisites are required

#### Arch

```bash
sudo pacman -S base-devel ncurses flex bison openssl libelf
```

#### Ubuntu

```bash
sudo apt-get install build-essential libncurses-dev bison flex libssl-dev libelf-dev
```

### Source code

The driver can be built in the kernel source tree or out of the kernel source tree. Below both methods are described.

Either way you will need a copy of the Linux kernel, the driver was updated to use a v5.4 kernel. To download and checkout the required version.

```bash
git clone https://github.com/torvalds/linux
cd linux
git checkout v5.4
```

### In tree

To build the module as part of the kernel you simply need to place the modified driver into the kernel source tree and perform a standard kernel build.

```bash
rm drivers/net/wireless/ath/ath9k/*
cp -r $(THIS REPO)/* drivers/net/wireless/ath/ath9k/
```

or run the patch to patch the changes into the ath9k folder. Please note this should be done from the directory in which the `linux` folder is.

```bash
patch -sf -p0 < patches/ath9k.patch
```

### Config

You will need to prepare the kernel to run on the target system, this is done through a `.config` file and it can be extracted from your existing system.

#### Arch

```bash
zcat /proc/config.gz > .config
```

#### Ubuntu

```bash
cp /boot/config-$(uname -r) .config
```

You will then need to run `menuconfig` or try `olddefconfig` to synchronize the configs.

```bash
make menuconfig
```
or
```bash
make olddefconfig
```

### Make

You must then perform a normal kernel build and a kernel and module install

```bash
sudo make -j$(nproc) && sudo make modules_install -j$(nproc) && sudo make install -j$(nproc)
```

or to build just the ath9k driver

```bash
sudo make -j$(nproc) drivers/net/wireless/ath/ath9k/
```

reboot and check if your new kernel is installed, otherwise continue below.

### Booting from kernel (Not always necessary)

My test machines run Ubuntu so I will detail the process for an Ubuntu machine. Arch machines are dependent on the bootloader you have chosen and, as such, harder to detail.

Update your initframfs and your grub.

```bash
update-initramfs -c -k 5.4
update-grub
```

Set the kernel as the default for grub by modifying `/etc/default/grub`

```bash
sudo vim /etc/default/grub
```

Then add the following to the `GRUB_DEAFULT` line so that it looks like the following

    GRUB_DEAFULT="Advanced options for Ubuntu>Ubuntu, with Linux 5.4"

finally reboot and check the new kernel is being used with `uname -r`

### Out of tree build

An out of tree build means that you simply pass the kernel tree into make with the `-C` option. This is done using the `KDIR` variable, set it appropriately.

As the Ath9k driver has the include structure of a potato it is challenging to build it out of tree as the driver has absolute include paths that traverse higher than the ath9k driver. One solution is to build the entire ath folder or to [patch](patches/ath.patch) the ath folder to fix up the include fro the ath9k driver. Sorry other ath driver but I have better things to do with my time.

```bash
export KDIR=$HOME/linux # Assuming linux was cloned into home
sudo make -C $KDIR M=$KDIR/drivers/net/wireless/ath modules
```

or if you want to directly install the built module

```bash
sudo make -C $KDIR -M $KDIR/drivers/net/wireless/ath modules_install
```

and reboot.

### Module install script

[here](https://github.com/alxhoff/dotfiles/blob/master/random_scripts/build_ath9k.sh)

An old build script I made when testing the WANDS Ubuntu 14 implementation. It should give an outline of the manual process, although the kernel build system should handle this automatically as detailed above.

### Travis CI

The Travis CI config (`.travis.yml`) also provides the steps necessary.