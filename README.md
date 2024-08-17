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

## Working with I2C based sensors and other external devices

1. Install i2c-tools: `sudo apt install i2c-tools`
2. The 'big core' is exposed to three i2c interfaces (2, 3, and 4). I am guessing that i2c-1 is reserved for the small core.
```
debian@duos:~$ ls /dev/i2c-*
/dev/i2c-2  /dev/i2c-3	/dev/i2c-4
```
3. The I2C pins are exposed on Header J3, see pinout here https://milkv.io/docs/duo/getting-started/duos#duos-gpio-pinout
4. Scanning and detecting I2C devices: I connected a BMP280 sensor to I2C-2 and did a `i2cdetect`
```
debian@duos:~$ sudo i2cdetect -y -r 2
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --
```
And found nothing :( TBC. Tried with I2C-4 and got same outcome :(
I think the issue has to due with pin-multiplexing - not sure. My working theory as that i2c has not been enabled on the target i2c pins. Unfortunately `duo-pinmux` on the Debian image does not appear to work correctly, it is printing a pinout that seem to match the Milk V Duo (64MB) rather than the Milk V Duo S. When I run `duo-pinmux` using the Milk V provided Buildroot image, I get very different pinout that seems to match the actual pinout on the Duo S.

I can manually get a working version of duo-pinmux by doing the follow

1. Copy `duo-pinmux` and `ld-musl-riscv64v0p7_xthead.so.1` from the latest Milk V buildroot (1.2) image (`/usr/bin/duo-pinmux` and `/lib/ld-musl-riscv64v0p7_xthead.so.1`), to the corresponding debian folders
2. Run duo-pinmux and see correct pinout
```
debian@duos:~$ sudo ./duo-pinmux -p
Pinlist:
A16
A17
...
C15
C12
C13
```
3. The pinout can now be updated with 'Milk V Buildroot' version of duo-pinmux and the I2C device is detected!
```
debian@duos:~$ sudo ./duo-pinmux -w B20/IIC4_SCL
pin B20
func IIC4_SCL
register: 3001158
value: 7

debian@duos:~$ sudo ./duo-pinmux -r B20
B20 function:
[ ] VI2_D_1
[ ] VI1_D_1
[ ] VO_D_14
[ ] B20
[ ] RMII0_RXDV
[ ] IIC3_SDA
[ ] PWM_3
[v] IIC4_SCL

register: 0x3001158
value: 7

debian@duos:~$ sudo ./duo-pinmux -w B21/IIC4_SDA
pin B21
func IIC4_SDA
register: 300115c
value: 7

debian@duos:~$ sudo ./duo-pinmux -r B20
B21 function:
[ ] VI2_D_0
[ ] VI1_D_0
[ ] VO_D_13
[ ] B21
[ ] RMII0_TXCLK
[ ] IIC3_SCL
[ ] WG1_D0
[v] IIC4_SDA

register: 0x300115c
value: 7

debian@duos:~$ sudo i2cdetect -y -r 4
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- 76 --
```
Works!

### ADS1015

```
# add user to i2c group to have read / write access
sudo usermod -a -G i2c debian
# setup python venv
mkdir pyvenv
cd pyvenv
python3 -m venv .venv
source .venv/bin/activate
```
inside the python venv, do
```
pip install Adafruit-ADS1x15
nano ./venv/lib/python3.12/site-packages/ADS1x15.py # i2c = SMBus(1) -> i2c = SMBus(4)
```
then i created a little python script `simple_ads.py`
```
import time
import ADS1x15

ADS = ADS1x15.ADS1015(4)
ADS.setGain(ADS.PGA_4_096V)

while True :
    raw = ADS.readADC(0)
    print("{0:.3f} V".format(ADS.toVoltage(raw)))
    time.sleep(1)
```
running it produces the expected outcome
```
(.venv) debian@duos:~/projects/pyvenv$ python3 simply_ads.py 
2.547 V
2.547 V
2.547 V
```

## Working with GPIOs 
The OS exposes 5 seperate 'GPIO chips', i.e.
```
debian@duos:~$ sudo gpiodetect 
gpiochip0 [3020000.gpio] (32 lines)
gpiochip1 [3021000.gpio] (32 lines)
gpiochip2 [3022000.gpio] (32 lines)
gpiochip3 [3023000.gpio] (32 lines)
gpiochip4 [5021000.gpio] (32 lines)
```
each of these control a group of GPIOs. 
Refere also to the Duo S documentation and pinout (https://milkv.io/docs/duo/getting-started/duos#gpio-pin-mapping). 

**Care should taken when working with GPIOs as many be used for system critical functions and manipulating the wrong GPIO may screw up your system. see also https://www.udoo.org/forum/threads/gpio-permissions-for-libgpiod-sudo-or-not.32453/ - Also note that some PINs are running at 3.3V logic level and others at 1.8V - be careful**

In the following I will try to expose 4 GPIOs from `XGPIOA` exposed on J3 that we can worked with in a safe way at 3.3V logic level. I have selected A18, A19, A20, A28 as these are already by default set to GPIO state in pinmux and do not appear to be used by anything else.

### GPIO pin mapping

| GROUP | ADDR | PORT | CHIP | NUM | NAME | START |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| gpio0 | gpio@03020000 | porta | gpiochip0 | 480-511 | XGPIOA | 480 - XGPIOA[0] |
| gpio1 | gpio@03021000 | portb | gpiochip1 | 448-479 | XGPIOB | 448 - XGPIOB[0] |
| gpio2 | gpio@03022000 | portc | gpiochip2 | 416-447 | XGPIOC | 416 - XGPIOC[0] |
| gpio3 | gpio@03023000 | portd | gpiochip3 | 384-415 |  |  |
| gpio4 | gpio@05021000 | porte | gpiochip4 | 352-383 | PWR_GPIO | 352 - PWR_GPIO[0] |

![Screenshot from 2024-08-17 10-49-17](https://github.com/user-attachments/assets/372002f9-3a9c-4e39-aec6-4621cd294433)

### Setting GPIOs 
Example A28 (gpiochip0 line 28)
```
debian@duos:~$ sudo gpioset gpiochip0 28=1
debian@duos:~$ sudo gpioset gpiochip0 28=0
```
To enable ´$USER´ (e.g. `debian` in my case) to have read/write access to GPIO group 0, I did
```
sudo addgroup gpio
sudo usermod -a -G gpio debian
sudo chown root.gpio /dev/gpiochip0
sudo chmod g+rw /dev/gpiochip0
```
The members of group `gpio` can now read/write to the GPIOs in gpio0 (i.e. `gpioset gpiochip0 28=0` without root permissions)
