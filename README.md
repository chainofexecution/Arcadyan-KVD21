# Hardware Hacking T-Mobile's 5G Home Internet Gateway
This repository will serve as a knowledge base for everything I've learned about the T-Mobile 5G Gateway so others can start off not completely in the dark like I had to. 

I've seen alot of posts on the FastMile variant manufactured by Nokia (a.k.a the Trashcan) but not much on the KVD21 from Arcadyan. I'm entirely still a novice at hardware hacking and will most likely end up bricking this thing but I work in the back of a T-Mobile doing repairs and get abandoned or devices for recycle all the time and want to try my hand at hardware hacking the exotic and uncommon devices I come across in my field of work.

Basic information and a device usage guide can be found here:

https://www.t-mobile.com/support/devices/get-to-know-your-arcadyan-kvd21-gateway

I've decided to structure entries in this journal by date as I'm going to be working mostly on this project (with one or two backup projects started incase I fry my gateway) as I want something eye catching to show off when I go to DEFCON for the first time in August later this year and having root on a 5G femtocell sandwiched to a home internet router that is a major sales focus for a major US carrier would be eye catching to say the least.

Due to the above mentioned change, and the fact that I have already done a good amount of research on the device prior to making this repository, there will be one large starting entry that will serve as the teardown and atleast two entries per week thereafter with a final long entry (provided I actually get root and dont fry the gateway) for the reception of the project following DEFCON 30.

## The Teardown and Initial Research

Also, all information, files, and other reference materials will eventually make their way on to the repository. The delay will be quite some time though as I have to ensure not to improperly disclose security vulnerabilities and purge personal or sensitive information from the materials and am currently extremely pressed for time so I can't spare much to reviewing material before publication.

The tear down of the device was quite easy after I figured out the right ammount of pressure to apply to the area holding the plastic clips together without breaking the clips.

With the cover off we can see an array of antennas and a screen with some buttons surrounding two boards that are sandwiched together.

![IMG_20220518_210400](https://user-images.githubusercontent.com/92492482/175692799-b68fd6c1-a8f9-4520-a50d-39958ae366f3.png)

![IMG_20220518_210413](https://user-images.githubusercontent.com/92492482/175694122-59f8567f-6b46-49eb-967e-7f1ea05e275c.png)

![IMG_20220518_210526](https://user-images.githubusercontent.com/92492482/175694199-8049059e-754c-4587-a53c-4880693781cb.png)

We can also see there is what looks like a UART port on the edge of one of the central PCBs.

![IMG_20220518_210514](https://user-images.githubusercontent.com/92492482/175694322-6ba9bcff-d275-49ff-9e5e-a8e5f327c623.png)

Using a multimeter to monitor voltages while the device is booting I see there is 0.4v on the pin by the arrow symbol, followed by 0v, 1.8v, and 0v. The pin farthest from the arrow symbol was verrified to be a ground pin by checking for continuity between the pin and a ground plane. The two pins in the middle are fluctuating 1.8v on the left and 0v stable on the right which indicates TX/RX respectively.

I soldered headers on to the 4 pin wells so I can get a reliable connection with an FTDI I did this while drunk so it looks horrible. 😅 

![IMG_20220623_124556](https://user-images.githubusercontent.com/92492482/175782661-2ce847a4-dfaa-4084-92ad-9449cf00d1ca.png)

This is the pinout for the UART connection I used with my Adafruit FT232H:

![IMG_20220623_124604](https://user-images.githubusercontent.com/92492482/175780366-adf2a95a-f00f-496f-b88c-e02159b5c263.png)

Pin    | Voltage | Voltage Flux | Pin Type | FT232H Pin
-------|---------|--------------|----------|------------
BLUE   | 0.4v    | Unstable     | UNKNOWN  | Unused
GREEN  | 0v      | Stable       | TX       | D1
YELLOW | 1.8v    | Unstable     | RX       | D0
ORANGE | 0v      | Stable       | GND      | GND

When we open up a tty serial on the FT232H and power the device on we get what appears to be a bootloader log that stops outputting shortly after the linux kernel starts. We cannot transmit over this tty but some valuable information was obtained from the bootloader log.

We also have some interesting bits of data at the start of the log that don't seem to be garbage from an improper baud rate or something similar. 

![Screenshot from 2022-06-25 13-09-12](https://user-images.githubusercontent.com/92492482/175783825-1150fc69-e7e5-468d-9000-4a8c082a1fe7.png)

When the data was separated from the text and opened in a hex editor like xxd we see that there are bits of 0x80 and 0x00 following a brief preamble of 0x10000000 and 0xFF008800.

![tty_start_bytes_xxd](https://user-images.githubusercontent.com/92492482/176510915-48377904-0353-4d13-9f69-6dc97c5970f6.png)

Moving on to the actual bootloader log we can see a return string entry from  the function dump_cmdline() that provides us with several key pieces of info: 

```
void dump_cmdline(void *):75: cmdline len=395, str="
  init=/etc/preinit
  root=/dev/mmcblk0p28
  rootwait
  loglevel=8
  androidboot.selinux=permissive
  androidboot.hardware=mt6890
  initcall_debug=1
  page_owner=on
  ramoops.mem_address=0x42880000
  ramoops.mem_size=0xe0000
  ramoops.pmsg_size=0x10000
  ramoops.console_size=0x40000
  mtk_printk_ctrl.disable_uart=0
  bootslot=a
  androidboot.hardware=mt6890
  bootprof.pl_t=1384
  bootprof.logo_t=0
  bootprof.lk_t=7882
"
```

* The gateway is running Android and we can use ADB once we know how to get the gateway into fastboot or download mode. 
* The modem processor is a Mediatek MT6890/MT6880 T75 based modem SoC package and we will verify this and narrow down the SoC package vendor during a more complete teardown later on. 
* SELinux is set to permissive. 🤣 This means once we have shell access on our gateway, priveledge escalation should be trivial despite Android being the target OS. 

# Week 2 - Jun 29th - Trust But Verify

We start off week two with a bootloader log from a UART port that told us the cpu was an mt6890.
I looked up the model number and found out from a marketing handout that chip is actually part of an SoC package called the T75 marketed for 5G IoT.


I wanted to tear down the device further anyways to thouroughly document the hardware so I will also verify the SoC package during the process.



