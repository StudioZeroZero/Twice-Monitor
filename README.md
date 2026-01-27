# 💓 Twice Monitor 💓 | A Low-Profile Ambient Display 
### An Updated Blueprint for Dedicated, Low-Power Media Kiosks (2026)

The Twice Monitor is an "ambient display" engineered to bring intentionality to desk and wall setups. Originally developed to automate the viewing experience of variety shows like Time to Twice, the device functions as a standalone, low-profile kiosk that autoplays curated media the moment it powers on.

### Why Did I Make This?

Hi! I'm surprised you found this, my name is Amit and I started this project because I am going to my very first TWICE concert in March and am so excited- but my room really doesn't reflect that interest! They say your room should is a reflection of who you are, and so I had this idea to have this digital decoration that could show off anything I've been interested in :D

It took over a month to find the resources/non-retired drivers needed to complete this project, but now I get to enjoy having this little device with me while I work. While I love the device, it really shouldn't have taken that long to make. And so- I want this page to be an all-in-one stop for dedicated customizable screens on a budget. To do so, I will be including everything I worked on for this project- from things like purchasing the materials and programming, to wiring diagrams and 3D printing. 

### **Key Features:**
- Kiosk-Mode Automation: Custom-configured Linux environment that bypasses the desktop GUI to launch directly into a Chromium-based kiosk mode for zero-latency interaction.

- Versatile Mounting: Designed with a lightweight footprint (approx. 80g / .2 lbs) optimized for non-destructive Command-strip wall mounting, balancing shear force requirements with thermal performance.

- Accessible Design: Optimized software to run perfectly on affordable, low-power hardware. Capability of smooth 1080p video playback and fast website loading without needing to buy a high-end display or a pricey Raspberry Pi 4.

- Universal Media Compatibility: Utilizing a custom Python/Bash backend to handle autoplay protocols for HTML5 video, YouTube, and dynamic web dashboards.


<img width="400" height="400" alt="IMG_5129-fotor-20260126185451" src="https://github.com/user-attachments/assets/7a900a8b-ba0e-4309-90da-aa6147a484d5" />
<img width="400" height="400" alt="IMG_5134-fotor-20260126185536" src="https://github.com/user-attachments/assets/5fe82bb7-fab6-4925-bc8e-68653ea117f6" />

# A Start-to-Finish Engineering Guide to Ambient Displays 

---
> [!NOTE]
> This guide uses my personal **"Twice Monitor"** build as a case study, but the instructions and CAD files are 100% applicable to any dedicated screen project.
## 💰 Bill of Materials (BOM)

Tools/Equipment (*Most universities/maker spaces provide these at a low/zero cost*):
- 3D Printer
- Soldering Iron
- Wire Cutter
- Wire Stripper
- Pliers

| Category | Item | Role | Value Choice | ~Price Range |
| :--- | :--- | :--- | :--- | :--- |
| **Compute** | Raspberry Pi Zero 2 | System Brain / Media Controller | Selected for optimal performance-to-price ratio. | %15 - $28 |
| **Display** | 2.4" IPS LCD Panel | Color 320x240, 2.8 inch, ILI9341 Controller | Sourced as a raw panel to avoid "branded" monitor markup. | $2 - $10 |
| **Power** | Micro USB Cable | Stable current for Pi & Screen | High-efficiency rating to prevent thermal waste. + Desired cord Length | $1 - $5 |
| **Chassis** | PLA | Structural support | 3D Print file provided in this github to mount the Pi Zero 2 directly to the ILI9341 Display | Free Download |
| **Mounting** | 3M Command Large Strips | Wall attachment | Apartment friendly mounting | ~$15 |
| **Storage** | 16GB MicroSD Card (with adapter/sd card reader if you don't already have one) | OS & Script storage | High-speed (Class 10) for fast boot times at low cost. | $5 - $10 |
| **Quality of Life Adapters (Optional if familiar with headless SSH setups)** | Mini HDMI -> HDMI adapter & Micro USB -> Type-A USB Adapter (to plug in a keyboard)| Temporary display and peripherals| For a one-time setup before we can control the Pi from our computer via SSH) | $5 - $10 |
| **Wiring Troubleshooting** | RPI Zero 2 Header & Jumper Wires | Temporary tools for troubleshooting wiring problems | For a one-time setup before we can solder the wire directly | $5 - $10 |
---


## 💻 Setting up the Pi

1. Download the Raspberry Pi Imager from the website and run the application.
- https://www.raspberrypi.com/software/

2. Select Diet Pi OS (OS -> "Other general-purpose OS" -> DietPi).

3. Insert SD card and select it as the the desired storage device. Flash the SD card.
 
4. After flashing, we need to enable Wi-Fi (dietpi.txt). While the SD card is still in your PC, open the card's folder and edit these two files:

- Find the file named dietpi.txt, open it, and make sure these two lines look exactly like this. This tells the Pi to ignore the Ethernet port and "listen" for Wi-Fi instead.

```

AUTO_SETUP_NET_ETHERNET_ENABLED=0

AUTO_SETUP_NET_WIFI_ENABLED=1

```

- Next find the file dietpi-wifi.txt, open it, and fill in your Wi-Fi info at the very top (under Entry 0):

```

aWIFI_SSID[0]='Your_Network_Name' //(Keep the single quotes!)

aWIFI_KEY[0]='Your_Password'

```

5. Insert the SD card into the Pi. After boot, type the following code line by line. (Use the adapters as needed to see and interact with the Pi)

```

// These lines of code will install the browser Chromium (to run the websites) and have it boot automatically on startup.

dietpi-software install 113
dietpi-autostart
select 11 : Chromium - Dedicated use without desktop

```

6. Here we will get an option to insert a link to our desired website. To get a youtube video to load automatically, we need to add the following to the end of the link: ?autoplay=1&mute=1

```

//example paste link for all of TTT
https://www.youtube.com/watch?v=7RIbfPbxu8M&list=PL5hjFu4TjrW2_k2HiWjf7bdr4vQNzKoQH&index=1&t=394s?autoplay=1&mute=1

```

7. On startup, sometimes Chromium will have pop ups that can't be removed without a mouse. To bypass "restore tabs" pop up on reboots, type the following line by line:

```
//sudo nano is a command used to access/edit a file of a declared directory
sudo nano /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh

// add these under #!/bin/dash
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences

```

8. For launching in a clean fashion (no scroll bar, autoplay, no cursor, max screen visibility) we need to also edit this line in /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh.

```
sudo nano /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh

// find the CHROMIUM_OPTS line of code and change accordingly
// note the "--window-size" and "--window-position" code will likely vary if you are using a display other than the 2.4" ILI9341
CHROMIUM_OPTS="--nocursor --no-memcheck--noerrdialogs --start-fullscreen --hide-scrollbars --autoplay-policy=no-user-gesture-required --start-maximized --force-device-scale-factor=.75 --high-dpi-support=1 --device-scale-factor=.75 --window-size=${RES_X:-320},${RES_Y:-240} --window-position=-35,40"

```

9. To display properly, we also need to change the Chromium resolution in /boot/dietpi.txt:

```
sudo nano /boot/dietpi.txt
//find the line reffering to the chromium resolutions and change according to your display ex: {RES_X:-320},${RES_Y:-240}

```

10. Now to get the actual video playback to fullscreen, we are going to use an extension called Enhancer for YouTube™ and enable the following settings:
- Extension Link: https://chromewebstore.google.com/detail/enhancer-for-youtube/ponfpcnoihfmfllpaingbgckeeldkhle?hl=en-US&utm_source=ext_sidebar
- Hide Shorts
- Hide related videos
- Hide chat
- Hide comments
- Automatically hide info cards and end screens
- Automatically expand the video player
- Use the available space based on the viewport dimensions to expand the video player

11. To get rid of pink video playback, disable graphic acceleration in chromium settings.

With this, you can now have your favorite be automatically ran on your monitor! Next, we will setup the SPI display for a more low-profile setup :)

## 🔌 Setting up the ILI9341 Display

1. Wire the ILI9341 to the RPI, following the diagram accordingly. (You can use jumper cables if you are not comfortable with soldering!)

<img width="720" height="638" alt="Screenshot 2026-01-26 162405" src="https://github.com/user-attachments/assets/a74b4a7b-fe69-4050-8367-3b4c87dd39b2" />

#### ILI9341 Display -> RPI Zero 2 Wiring Diagram

| Display Pin | Pi Physical Pin | Function |
| :--- | :--- | :--- |
| VCC | 1 | 3.3V |
| GND | 6 | Ground |
| CS | 24 | GPIO 8 |
| RESET | 13 | GPIO 27 |
| DC | 15 | GPIO 22 |
| MOSI | 19 | GPIO 10 |
| SCK | 23 | GPIO 11 |
| LED | 17 | Backlight |
| MISO | 21 | GPIO 9 |

> [!TIP]
> If you are soldering, I would reccomend to remove the plastic off the headers with a pliers and then remove them with the soldering iron- made it much easier for me!

2. Waveshare32 software for the RPI to communicate with the Display.

```

sudo apt update && sudo apt install git -y
cd LCD-show/
sudo ./LCD28-show

dietpi-config
display
lcd addons
waveshare32

```

3. For better playback, disable overscan in display config.

```

dietpi-config
//Navigate to 'Display Options' -> 'Overscan' -> 'Disabled' -> 'Exit and Reboot'

```

## 🛠️ Assembly

1. Download, slice and 3D print the following files:
- [Front Shell STEP File](cad/rpi_front_shell.STEP)
- [Back Shell STEP File](cad/rpi_back_shell.STEP)
- [SPI Adapter STEP File](cad/rpi_spi_adapter.STEP)

<img width="500" height="500" alt="angle 2" src="https://github.com/user-attachments/assets/bf98a281-6b02-47c5-81aa-f92a2c7f9e76" />
<img width="500" height="500" alt="Screenshot 2026-01-26 171851" src="https://github.com/user-attachments/assets/57b9236e-87a5-45fb-a1e8-6c169679b0fe" />

> [!TIP]
> If you have never 3D Printed something before, this is the simplest way to go about printing these files:
> Download Files -> Download and open Orca Slicer -> Drag in Step Files -> Click 'Slice" -> Save exported file to flashdrive -> plug into 3D printer and print

2. Use M2 nuts and bolts to first assemble the RPI to the 3D printed file 'rpi_spi_adapter' (Place an approiately sized M2 Bolt in through the top side, using a nut to fastening both the RPI and the SPI together).

3. Insert the now fastened RPI and SPI into the 'Front Shell', repeating the same fastening process in the last step to connect the 'Front Shell' to the outer holes of the 'SPI Adapter'.

4. Repeat to fasten the 'Back Shell' to the 'Front Shell'.

5. You can either leave the device standing as is, or cut a strong command strip along the extruded part of the back casing to mount on a wall.

## Congratulations!
You have built yourself a neat automated monitor to display whatever you desire. What will you use yours for?

![IMG_5086-fotor-2026012619618](https://github.com/user-attachments/assets/fee169b2-9019-4a0f-a553-c47aa5f3a30a)

## Future Iterations
* **Portable Power (LiPo Integration):** Incorporate a Lithium Polymer battery and charging circuit  to transition from a tethered device to a portable ambient display.
* **Tactile User Interface:** Integrate a rotary encoder for volume/brightness control and mechanical switches for physical "Play/Pause" and "Skip" functions.
* **Unibody Enclosure:** Iterate the SOLIDWORKS design from a two-part shell to a minimalist, snap-fit unibody chassis with internal cable routing and hidden mounting points.
* **Thermal Management:** Optimize the internal geometry to utilize a "Passive Chimney Effect," increasing airflow across the SoC without the need for active cooling (fans).
* **Environmental Sensing:** Add a PIR motion sensor to toggle the display based on room occupancy, significantly reducing power consumption and extending the LCD backlight lifespan.
* **Auto-Dimming:** Use a photoresistor (LDR) to automatically adjust screen brightness based on ambient light levels, ensuring the monitor remains non-intrusive at night.

## 💖 Special Thanks
* **JYP Entertainment / TWICE:** For the endless "Time to Twice" content that fueled this build.
* **The DietPi Community:** For providing the lightweight OS that makes this project possible on affordable hardware.
