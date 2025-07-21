# edid-inject-for-fedora-wayland
How to inject EDID in Fedora 42 Gnome Wayland.
I suceeded in doing this using AW EDID Editor. I also tried CRU, but it kept on breaking my monitor configuration.
For this you need monitor-get-edid, monitor-parse-edid and aforementioned AW EDID Editor.
Command order: 
1. sudo dnf install monitor-edid
2. monitor-get-edid > 'name-of-your-choice'.bin # Without apostrophes! Or with them, but to the very end!
3. cat <name-of-your-choice>.bin | monitor-parse-edid # This command should output 
	Name: W2242
EISA ID: GSM5678
EDID version: 1.3
EDID extension blocks: 0
Screen size: 49.0 cm x 32.0 cm (23.04 inches, aspect ratio 3/2 or 16/10 = 1.53)
Gamma: 2.2
Digital signal
Max video bandwidth: 150 MHz

	HorizSync 30-83
	VertRefresh 56-75

	# Monitor preferred modeline (59.9 Hz vsync, 64.7 kHz hsync, ratio 16/10, 87 dpi)
	ModeLine "1680x1050" 119 1680 1728 1760 1840 1050 1053 1059 1080 -hsync +vsync

	# Monitor supported modeline (60.0 Hz vsync, 65.3 kHz hsync, ratio 16/10, 87 dpi)
	ModeLine "1680x1050" 146.25 1680 1784 1960 2240 1050 1053 1059 1089 +hsync -vsync

In my case this was the output. If you also have Name, Screen size, and other information you are good to go with proceeding on to AW EDID Editor. 
Pay special attention to Max video bandwidth, in my case it's 150 MHz. This tells how many MHz your monitor can accept.
  Under X11 I used simple xrandr command on startup which would set this mode for me. Under wayland it's not possible.
Open the file you've just created, proceed to the last tab 'Detailed Descriptor' in Editor, stay on Block 1 interface - interface is very user friendly, so it's hard 
to mess something up. Now you see 'Detailed timing'. Click on CVT Format wizard and, in my case, set 'Reduced blankings' to ON.
Tweak Pixel Clock to the limit of bandwidth (150 MHZ). It will self-adjust to use the maximum refresh rate.
Now in top-right corner select 'File', then 'Save as' and choose a name for your new bin file.
5. cat 'name-of-your-choice-AW'.bin | monitor-parse-edid # Check whether the configuration is set, if it's alright, then you can go further. 
# Pay special attention To Modeline "YYYYxZZZZ" QQQ - these QQQ are your set bandwidth, they should not exceed the Max video bandwidth at any means!
# Now check hsync vsync - if those are set as they should, in my case it's +hsync -vsync, then go further
5. sudo mkdir /lib/firmware/edid/ #Create a directory where you will put this bin file 
6. sudo cp 'name-of-your-choice-AW'.bin /lib/firmware/edid/ # Copy AW EDID Editor output to this directory you previously had created.
7. xrandr --verbose # Use this to identify which port your monitor utilizes.
8. sudo nano /etc/default/grub # Open the grub config and copy this: drm.edid_firmware=edid/'name-of-your-choice-AW'.bin video='port name':e #Without apostrophes for video='port name', obviously.
#If you have finished the step number 8, you can now proceed to step number 9
9. sudo grub2-mkconfig -o /boot/grub2/grub.cfg # Update your grub to actually inject the EDID.
9. reboot # Initialize the change. After rebooting, plug out your monitor and plug it back in if your screen isn't outputting anything, this helped me. 
