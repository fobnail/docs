# Booting Ubuntu from USB by using kexec

This document describes how to boot Ubuntu 22.04 LTS flashed on USB memory with
`kexec`. That gives the possibility to quick boot (without hardware
reinitialization) of destination system after successful attestation.

## Requirements
- To work with `kexec` you need to have installed `kexec-tools` on the system
image and enabled `CONFIG_KEXEC` in the kernel configuration. It was introduced
in 0.2.0 `meta-fobnail` version.

- Pendrive with Ubuntu 22.04 LTS plugged to apu2

## Running

Boot to the linux shell on `meta-fobnail` image. Pendrive should be visible as a
`/dev/sdX`

```
# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    1 14.4G  0 disk
|-sda1        8:1    1  3.4G  0 part
|-sda2        8:2    1  4.1M  0 part
|-sda3        8:3    1  300K  0 part
`-sda4        8:4    1   11G  0 part
mmcblk0     179:0    0   29G  0 disk
|-mmcblk0p1 179:1    0   64M  0 part /boot
`-mmcblk0p2 179:2    0  512M  0 part /
```

Create a USB folder, mount and enter to the device:

```
# mkdir /mnt/usb
# mount /dev/sda /mnt/usb
mount: /mnt/usb: WARNING: source write-protected, mounted read-only.
# cd /mnt/usb
```

Load new Ubuntu kernel:

```
# kexec -l casper/vmlinuz --initrd=casper/initrd --command-line="$( cat /proc/cmdline )"
# kexec -e
[   81.534986] kexec_core: Starting new kernel
[    0.000000] Linux version 5.15.0-25-generic (buildd@ubuntu) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #25-Ubuntu SMP Wed Mar 30 15:54:22 UTC 2022 (Ubuntu 5.15.0-25.25-gene)
[    0.000000] Command line: BOOT_IMAGE=/bzImage root=/dev/mmcblk0p2 console=ttyS0,115200 earlyprintk=serial,ttyS0,115200
```

After a minute you should be able to login into the Ubuntu shell:

```
[  OK  ] Started Serial Getty on ttyS0.
[  OK  ] Reached target Login Prompts.
         Starting Set console scheme...

Ubuntu 22.04 LTS ubuntu ttyS0

ubuntu login: ubuntu
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-25-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ubuntu:~$ uname -a
Linux ubuntu 5.15.0-25-generic #25-Ubuntu SMP Wed Mar 30 15:54:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
ubuntu@ubuntu:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04 (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```
