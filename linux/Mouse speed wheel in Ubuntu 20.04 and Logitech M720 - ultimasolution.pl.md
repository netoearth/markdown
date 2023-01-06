-   [Option 1: xinput](https://www.ultimasolution.pl/mouse-speed-wheel/#option-1-xinput "Option 1: xinput")
-   [Option 2 : imwheel – confirmed working on 20.04](https://www.ultimasolution.pl/mouse-speed-wheel/#option-2-imwheel-confirmed-working-on-2004 "Option 2 : imwheel – confirmed working on 20.04")
-   [Option 3: Solaar](https://www.ultimasolution.pl/mouse-speed-wheel/#option-3-solaar "Option 3: Solaar")

## Option 1: xinput

```
xinput list
```

Now pick the ID of your mouse there, and list its current settings:

$ xinput list-props <device-id>

Device 'Logitech M720 Triathlon':

Coordinate Transformation Matrix (181): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000

libinput Natural Scrolling Enabled (316): 0

libinput Natural Scrolling Enabled Default (317): 0

libinput Scroll Methods Available (318): 0, 0, 1

libinput Scroll Method Enabled (319): 0, 0, 0

libinput Scroll Method Enabled Default (320): 0, 0, 0

libinput Button Scrolling Button (321): 2

libinput Button Scrolling Button Default (322): 2

libinput Middle Emulation Enabled (323): 0

libinput Middle Emulation Enabled Default (324): 0

libinput Accel Speed (325): 0.900000

libinput Accel Speed Default (326): 0.000000

libinput Accel Profiles Available (327): 1, 1

libinput Accel Profile Enabled (328): 1, 0

libinput Accel Profile Enabled Default (329): 1, 0

libinput Left Handed Enabled (330): 0

libinput Left Handed Enabled Default (331): 0

libinput Send Events Modes Available (301): 1, 0

libinput Send Events Mode Enabled (302): 0, 0

libinput Send Events Mode Enabled Default (303): 0, 0

Device Node (304): "/dev/input/event19"

Device Product ID (305): 1133, 16478

libinput Drag Lock Buttons (332):

libinput Horizontal Scroll Enabled (333): 1

$ xinput list-props <device-id> $ xinput list-props 11 Device 'Logitech M720 Triathlon': Device Enabled (179): 1 Coordinate Transformation Matrix (181): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000 libinput Natural Scrolling Enabled (316): 0 libinput Natural Scrolling Enabled Default (317): 0 libinput Scroll Methods Available (318): 0, 0, 1 libinput Scroll Method Enabled (319): 0, 0, 0 libinput Scroll Method Enabled Default (320): 0, 0, 0 libinput Button Scrolling Button (321): 2 libinput Button Scrolling Button Default (322): 2 libinput Middle Emulation Enabled (323): 0 libinput Middle Emulation Enabled Default (324): 0 libinput Accel Speed (325): 0.900000 libinput Accel Speed Default (326): 0.000000 libinput Accel Profiles Available (327): 1, 1 libinput Accel Profile Enabled (328): 1, 0 libinput Accel Profile Enabled Default (329): 1, 0 libinput Left Handed Enabled (330): 0 libinput Left Handed Enabled Default (331): 0 libinput Send Events Modes Available (301): 1, 0 libinput Send Events Mode Enabled (302): 0, 0 libinput Send Events Mode Enabled Default (303): 0, 0 Device Node (304): "/dev/input/event19" Device Product ID (305): 1133, 16478 libinput Drag Lock Buttons (332): libinput Horizontal Scroll Enabled (333): 1

```
$ xinput list-props <device-id>
$ xinput list-props 11
Device 'Logitech M720 Triathlon':
        Device Enabled (179):   1
        Coordinate Transformation Matrix (181): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        libinput Natural Scrolling Enabled (316):       0
        libinput Natural Scrolling Enabled Default (317):       0
        libinput Scroll Methods Available (318):        0, 0, 1
        libinput Scroll Method Enabled (319):   0, 0, 0
        libinput Scroll Method Enabled Default (320):   0, 0, 0
        libinput Button Scrolling Button (321): 2
        libinput Button Scrolling Button Default (322): 2
        libinput Middle Emulation Enabled (323):        0
        libinput Middle Emulation Enabled Default (324):        0
        libinput Accel Speed (325):     0.900000
        libinput Accel Speed Default (326):     0.000000
        libinput Accel Profiles Available (327):        1, 1
        libinput Accel Profile Enabled (328):   1, 0
        libinput Accel Profile Enabled Default (329):   1, 0
        libinput Left Handed Enabled (330):     0
        libinput Left Handed Enabled Default (331):     0
        libinput Send Events Modes Available (301):     1, 0
        libinput Send Events Mode Enabled (302):        0, 0
        libinput Send Events Mode Enabled Default (303):        0, 0
        Device Node (304):      "/dev/input/event19"
        Device Product ID (305):        1133, 16478
        libinput Drag Lock Buttons (332):       
        libinput Horizontal Scroll Enabled (333):       1

```

then change the settings like so where Evdev scrolling distance \[vertical\] \[horizontal\] \[dial\]

$ xinput set-prop <device-id\> 'Evdev Scrolling Distance' 1 3 5

$ xinput set-prop <device-id> 'Evdev Scrolling Distance' 1 3 5

```
$ xinput set-prop <device-id> 'Evdev Scrolling Distance' 1 3 5

```

where the combination of the last three numbers is mouse-dependent:

– first number, the direction of scrolling (minus reverse)  
– second number, speed of scrolling somehow  
– third number, speed of scrolling somehow  
– Changing these values to bigger numbers means you scroll slower (AgentME).

## Option 2 : imwheel – confirmed working on 20.04

1.  Install
    
    imwheel
    
    `imwheel` and adjust (to make things work):

-   Run
    
    sudo apt install imwheel
    
    `sudo apt install imwheel`
-   Run
    
    curl -s http://www.nicknorton.net/mousewheel.sh > wheel.sh
    
    `curl -s http://www.nicknorton.net/mousewheel.sh > wheel.sh`
-   Using the slider, adjust the scroll speed ‘multiplier’. (I like it on 4/5)

Control\_L, Up, Control\_L|Button4

Control\_L, Down, Control\_L|Button5

Shift\_L, Up, Shift\_L|Button4

Shift\_L, Down, Shift\_L|Button5

INFO: imwheel(pid=300395) killed.

INFO: imwheel started (pid=305311)

./wheel.sh ".\*" None, Up, Button4, 5 None, Down, Button5, 5 Control\_L, Up, Control\_L|Button4 Control\_L, Down, Control\_L|Button5 Shift\_L, Up, Shift\_L|Button4 Shift\_L, Down, Shift\_L|Button5 INFO: imwheel(pid=300395) killed. INFO: imwheel started (pid=305311)

```
./wheel.sh
".*"
None,      Up,   Button4, 5
None,      Down, Button5, 5
Control_L, Up,   Control_L|Button4
Control_L, Down, Control_L|Button5
Shift_L,   Up,   Shift_L|Button4
Shift_L,   Down, Shift_L|Button5
INFO: imwheel(pid=300395) killed.
INFO: imwheel started (pid=305311)
```

2.  Add
    
    imwheel
    
    `imwheel` as a startup application (to make things continue working after restart):

-   Open _Apps -> Startup Applications_
-   Add a new entry to the bottom of the list: _Name_\= `Wheel Scroll Speed`, _Command_\= `imwheel`, _Comment_\= `Activates wheel scroll speed fix on system startup` (or whatever you like)

## Option 3: Solaar

[https://pwr-solaar.github.io/Solaar/installation](https://pwr-solaar.github.io/Solaar/installation)

![](https://www.ultimasolution.pl/wp-content/uploads/2022/02/Screenshot-2022-02-12_1727.png)