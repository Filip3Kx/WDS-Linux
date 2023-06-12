# WDS-Linux
A configuration of Windows Deployment Services that will install Linux operating systems without breaking Windows installations.

The whole idea revolves around the fact that a boot wim image can be used as a menu entry in the Syslinux PXE boot menu.

## Prequesites
- [Syslinux](https://www.kernel.org/pub/linux/utils/boot/syslinux/) (I used 6.03. There could be dependency issues for some of the lib files between versions)
- Windows Server (I used 2016 and lower. Had trouble to make windows installations work on 2019)
- DHCP, WDS and NFS sharing are services that need to be installed or served from a different machine
- Linux Iso. In this tutorial you'll find out how to install:
  - Ubuntu
  - CentOS
  - OpenSuse
- Windows Iso. To check if windows installations are still functional

![Pasted image 20230529103646](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/3a56397f-bcb2-4546-bb38-5de0b1463f47)

## Syslinux
All needed files are in the [syslx](https://github.com/Filip3Kx/WDS-Linux/tree/main/syslx) directory of the repo. But if you want to use versions other than `6.03` here's what you'll need and where to find it:
- syslinux-6.03\bios\core\pxelinux.0
- syslinux-6.03\bios\com32\menu\vesamenu.c32
- syslinux-6.03\bios\com32\chain\chain.c32

And here are the dependencies of these files
- syslinux-6.03\bios\com32\elflink\ldlinux\ldlinux.c32
- syslinux-6.03\bios\com32\elflink\ldlinux\ldlinux.elf
- syslinux-6.03\bios\com32\libutil\libutil.c32
- syslinux-6.03\bios\com32\cmenu\libmenu\libmenu.c32
- syslinux-6.03\bios\com32\gpllib\libgpl.c32
- syslinux-6.03\bios\com32\lua\src\liblua.c32
- syslinux-6.03\bios\com32\lib\libcom32.c32

## WDS Configuration
After you've configured all of the services listed above we can start by replacing the boot files of WDS with the syslinux ones.

![image](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/3db80aeb-2806-4c62-b811-dfb313ce2a6a)

1. Dump all files from the syslx directory into the `boot/x64` folder in the directory tree of WDS
![image | 200](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/c7e4bc50-ac95-431e-8136-42368da4bda8)
2. Change name of `pxelinux.0` to `pxelinux.com`
3. Create directory `pxelinux.cfg`
4. Make a copy of `pxeboot.n12` and call it `pxeboot.0`
5. Make a copy of `abortpxe.com` and call it `abortpxe.0`
6. Create folder called `Linux`

You can place the [menu_ws](https://github.com/Filip3Kx/WDS-Linux/tree/main/menu_ws) file in the `pxelinux.cfg` directory and rename the file to `default`. **This is a sample menu that we will use for testing out windows installations (it doesn't have any linux menu entries).**

Now we can point the WDS to new boot files using these commands
```
wdsutil /set-server /bootprogram:boot\x64\pxelinux.com /architecture:x64
wdsutil /set-server /N12bootprogram:boot\x64\pxelinux.com /architecture:x64
```

## Windows installation setup
Open your Windows iso with some archiving tool. For example [7zip](https://www.7-zip.org/).

After you've opened the Iso we will need 2 files
- sources/install.wim
- sources/boot.wim

In the WDS configuration tool we can point to these 2 files as our install image and boot image
![Pasted image 20230529103520](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/ebe646cb-d9cc-47e9-a96d-29007139d9b8)
![Pasted image 20230529103556](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/51527416-58bd-4947-8092-88e6aa279321)

## Testing Windows installation
![Pasted image 20230529103716](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/6412f803-eb11-47da-9160-081400836788)
![Pasted image 20230529103430](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/d4e69197-8bdc-420b-bb54-2e42088b05d5)

## Linux installation setup
In the directory `.../boot/x64/Linux` we've made earlier we will make a subdirectory for every ISO we want to install and then point to it using the `default` menu file. For example i want to install Ubuntu so the path will looke like this `.../boot/x64/Linux/Ubuntu`. Now you can open the linux iso using [7zip](https://www.7-zip.org/) and extract all it's files into this path.

After that is done we will need to configure our folder to be an NFS share that will serve all the installation files.

![Pasted image 20230529103845](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/e345cc2b-6bf0-40d4-8f7d-f4b62f3d60b3)
![Pasted image 20230529103857](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/adea2da7-f42d-449f-b0e3-1e5e8fdaa3d0)

Be sure to remember about the filesystem permissions as well because they override any of the previous changes

![Pasted image 20230529103957](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/eb7155c3-4468-4783-962d-505c54dbd8b2)

With that you can replace the previous menu file in `pxelinux.cfg/default` with [menu_lx](https://github.com/Filip3Kx/WDS-Linux/tree/main/menu_lx). **Remember to rename it do default and be sure to open it and change the values for your configuration**

## Testing Linux Installation
![Pasted image 20230529104057](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/64a960ca-91ad-4e2c-9bc6-cdadcf43bb78)
![Pasted image 20230529105231](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/447af277-bce9-4b25-8664-3058f79f23e8)
![Pasted image 20230529105353](https://github.com/Filip3Kx/WDS-Linux/assets/114138650/b7f6b8d9-9057-44c2-a3bb-2b0f117c9765)

## Working iso's
- Ubuntu 22.04.2
```
APPEND boot=casper ip=dhcp netboot=nfs nfsroot=192.168.1.5:/Ubuntu initrd=/Linux/Ubuntu/casper/initrd
```
- Ubuntu 20.04.4
```
APPEND boot=casper ip=dhcp netboot=nfs nfsroot=192.168.1.5:/Ubuntu initrd=/Linux/Ubuntu/casper/initrd
```
- Ubuntu 20.04.6
```
APPEND boot=casper ip=dhcp netboot=nfs nfsroot=192.168.1.5:/Ubuntu initrd=/Linux/Ubuntu/casper/initrd
```
- CentOs 7
```
APPEND method=nfs:192.168.1.5:/CentOS initrd=/Linux/CentOs/initrd.img
```
- OpenSuse
```
 append root=/dev/nfs vga=0x314 initrd=/linux/OpenSuse/initrd showopts splash=silent install=nfs://192.168.1.5:/OpenSuse
```
