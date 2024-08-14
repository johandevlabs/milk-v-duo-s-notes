# milk-v-duo-s-notes
Notes on working with the milk v duo s linux

## Useful links
- https://github.com/Fishwaldo/sophgo-sg200x-debian
- https://community.milkv.io/t/how-to-get-the-operating-system-on-the-emmc/1634
- https://milkv.io/docs/duo/getting-started/duos


## Installing the Debian image on Duo S EMMC
I Followed this guide https://community.milkv.io/t/how-to-get-the-operating-system-on-the-emmc/1634

1. Using `GParted` or similar, prepare a msdos / FAT-32 formatted SD-card.
2. Go to https://github.com/Fishwaldo/sophgo-sg200x-debian and download the Debian Duo S for EMMC image (I used duos_emmc.zip from release v1.4.0)
3. Unzip files and copy to root directory of FAT 32 SD CARD
4. Insert the SD-card in DUO S and power on (connect to UART TTY interface to track progress - optional) and wait ~1 min for system to copied over automatically
5. Power off and remove SD card
6. Power on and system will boot into Debian (**Sid**)
7. Login in with debian / rv

## Getting started with Debian on Milk V Duo S
You will need terminal access to the Duo S. This can either be achieved directly using the UART interface or over the USB-C RNDIS network interface. I chose the later as it seems nicer to work with
1. Connect the Duo S to your host machine (e.g. laptop over USB-C cable).
2. On host: ``ip link`` to see if new RNDIS interface has been created. This might take 30 s as the Duo S boots up. In my case the interface was called `enx......` but it might be called `usb0`
RNDIS is essentially a network created by the Duo S and it will assign an IP to the host machine
3. Figure out what the IP of the Duo S is on the RNDIS netowrk. `hostname -I` and look for a new IP address, in my case the host was assigned `10.42.0.20` and guessed the Duo S to have `10.42.0.1`
4. Connect over SSH, i.e. `ssh debian@10.42.0.1`
5. Update password `passwd` and root password `su -` then `passwd root` and then `exit`
6. Connect to WiFi (e.g. `sudo nmtui`)

### Resizing the EMMC partition
By default, the Debian image will create a 776MB root partition, but the EMMC has 8GB. To resize, do the following
1. Inspect filesystem and disk (e.g. `df -h` and `lsblk`)
```
debian@duos:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       776M  553M  167M  77% /
devtmpfs        244M     0  244M   0% /dev
tmpfs           244M     0  244M   0% /dev/shm
tmpfs            98M  600K   97M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           244M     0  244M   0% /tmp
tmpfs            49M  4.0K   49M   1% /run/user/1000
debian@duos:~$ lsblk 
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0      179:0    0  7.3G  0 disk 
`-mmcblk0p1  179:1    0  7.3G  0 part /
mmcblk0boot0 179:8    0    4M  1 disk 
mmcblk0boot1 179:16   0    4M  1 disk
```
2. Resize the partiotion to fill out the whole EMMC disk (using `resize2fs` directed at the root partition).
```
debian@duos:~$ sudo resize2fs /dev/mmcblk0p1 
[sudo] password for debian: 
resize2fs 1.47.1 (20-May-2024)
Filesystem at /dev/mmcblk0p1 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mmcblk0p1 is now 1908735 (4k) blocks long.

debian@duos:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       7.2G  556M  6.4G   8% /
devtmpfs        244M     0  244M   0% /dev
tmpfs           244M     0  244M   0% /dev/shm
tmpfs            98M  600K   97M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           244M     0  244M   0% /tmp
tmpfs            49M  4.0K   49M   1% /run/user/1000
```

### Updating the system and installing additional software

I am bit concerned about using Sid rather than Stable or Testing. Care should be taken when upgrading or updating the system. Note to self: Inspect apt output before selecting `Y` and never use the `-y` option when running apt.

1. `sudo apt install neofetct`
```
debian@duos:~$ neofetch 
       _,met$$$$$gg.          debian@duos 
    ,g$$$$$$$$$$$$$$$P.       ----------- 
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux trixie/sid riscv64 
 ,$$P'              `$$$.     Host: Milk-V DuoS 
',$$P       ,ggs.     `$$b:   Kernel: 5.10.4-20240527-2+ 
`d$$'     ,$P"'   .    $$$    Uptime: 47 mins 
 $$P      d$'     ,    $$P    Packages: 412 (dpkg) 
 $$:      $$.   -    ,d$$'    Shell: bash 5.2.21 
 $$;      Y$b._   _,d$P'      Terminal: /dev/pts/0 
 Y$$.    `.`"Y$$$$P"'         CPU: (1) 
 `$$b      "-.__              Memory: 54MiB / 486MiB 
  `Y$$
   `Y$$.                                              
     `$$b.                                            
       `Y$$b.
          `"Y$b._
              `"""
```
2. `sudo apt install python3 python3-venv python3-pip`


