# Hardware Hacking T-Mobile's 5G Home Internet Gateway
This repository will serve as a knowledge base for everything I've learned about the T-Mobile 5G Gateway so others can start off not completely in the dark like I had to. 

I've seen alot of posts on the FastMile variant manufactured by Nokia (a.k.a the Trashcan) but not much on the KVD21 from Arcadyan. I'm entirely still a novice at hardware hacking and will most likely end up bricking this thing but I work in the back of a T-Mobile doing repairs and get abandoned or devices for recycle all the time and want to try my hand at hardware hacking the exotic and uncommon devices I come across in my field of work.

Basic information and a device usage guide can be found here:

https://www.t-mobile.com/support/devices/get-to-know-your-arcadyan-kvd21-gateway

The Teardown:

The tear down of the device was quite easy after I figured out the right ammount of pressure to apply to the area holding the plastic clips together without breaking the clips.

With the cover off we can see an array of antennas and a screen with some buttons surrounding two boards that are sandwiched together.

![IMG_20220518_210400](https://user-images.githubusercontent.com/92492482/175692799-b68fd6c1-a8f9-4520-a50d-39958ae366f3.png)

![IMG_20220518_210413](https://user-images.githubusercontent.com/92492482/175694122-59f8567f-6b46-49eb-967e-7f1ea05e275c.png)

![IMG_20220518_210526](https://user-images.githubusercontent.com/92492482/175694199-8049059e-754c-4587-a53c-4880693781cb.png)


We can also see there is what looks like a UART port on the edge of one of the central PCBs.

![IMG_20220518_210514](https://user-images.githubusercontent.com/92492482/175694322-6ba9bcff-d275-49ff-9e5e-a8e5f327c623.png)

Using a multimeter to monitor voltages while the device is booting I can see the pinout is as follows:

Pin    | Voltage | Voltage Flux | Pin Type
-------|---------|--------------|---------
BLUE   | 0.4v    | Unstable     | UNKNOWN
GREEN  | Ov      | Stable       | TX
YELLOW | 1.8v    | Unstable     | RX
ORANGE | 0v      | Stable       | GND

I soldered headers on to the 4 pin wells (while drunk so it looks horrible) and broke out my Adafruit FT232H. 

I have a special place in my heart for both Python and organizations that help empower women to learn/work in STEM so CircuitPython boards from Adafruit Industries is the natural choice.

When we open up a tty serial on the FT232H and power the device on we get what appears to be a bootloader log that stops outputting shortly after the linux kernel starts. 
We cannot type anything into this prompt but some valuable information was obtained from the bootloader log (which will be available in this repo)
