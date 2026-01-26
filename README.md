# :heartpulse:Twice-Monitor:heartpulse: 
A dedicated, low-profile ambient display system designed for seamless high-quality media integration.

AKA a device to automatically play my favorite variety show TTT (TIME TO TWICE) in the background :D

## About Me

Hi! I'm surprised you found this, my name is Amit and I started this project because I am going to my very first TWICE concert in March and am so excited- but my room doesn't reflect that interest! They say your room should is a reflection of who you are, and so my idea was to have a piece of digital decoration that could show off anything I've been interested in :D

It was much more difficult than I imagined to find the resources need to complete this project, but it's been so much fun having this little device accomany me while I work. And so- I want this page to be an all-in-one stop for anyone else wanting to make something like this (or a reminder for myself when I forget how I did any of this xD). To do so, I will be including everything- from things like purchasing the materials and programming, to wiring diagrams and 3D printing. 

## **Key Features:**
- Kiosk-Mode Automation: Custom-configured Linux environment that bypasses the desktop GUI to launch directly into a Chromium-based kiosk mode for zero-latency interaction.

- Versatile Mounting: Designed with a lightweight footprint (approx. 80g / .2 lbs) optimized for non-destructive Command-strip wall mounting, balancing shear force requirements with thermal performance.

- Accessible Design: Optimized software to run perfectly on affordable, low-power hardware. Capability of smooth 1080p video playback and fast website loading without needing to buy a high-end display or a pricey Raspberry Pi 4.

- Universal Media Compatibility: Utilizing a custom Python/Bash backend to handle autoplay protocols for HTML5 video, YouTube, and dynamic web dashboards.

# A Cohesive Guide to Setting Up A Youtube Auto Player Miniscreen

## 🛠️ Bill of Materials (BOM)

Tools/Equipment (*Most universities/maker spaces provide these at a low/zero cost*):
- 3D Printer
- Soldering Iron
- Wire Cutter
- Wire Stripper
- Pliers

| Category | Item | Role | Value Choice | ~Price Range |
| :--- | :--- | :--- | :--- | :--- |
| **Compute** | Raspberry Pi Zero 2 | System Brain / Media Controller | Selected for optimal performance-to-price ratio. | 15 - $28$ |
| **Display** | 2.4" IPS LCD Panel | Color 320x240, 2.8 inch, ILI9341 Controller | Sourced as a raw panel to avoid "branded" monitor markup. | 2 - $10 |
| **Power** | Micro USB Cable | Stable current for Pi & Screen | High-efficiency rating to prevent thermal waste. + Desired cord Length | 1 - $5 |
| **Chassis** | PLA | Structural support | 3D Print file provided in this github to mount the Pi Zero 2 directly to the ILI9341 Display | Free Download |
| **Mounting** | 3M Command Large Strips | Wall attachment | Apartment friendly mounting | ~$15 |
| **Storage** | 16GB MicroSD Card (with adapter/sd card reader if you don't already have one) | OS & Script storage | High-speed (Class 10) for fast boot times at low cost. | 5 - $10 |
| **Quality of Life Adapters (Optional if familiar with headless SSH setups)** | Mini HDMI -> HDMI adapter & Micro USB -> Type-A USB Adapter (to plug in a keyboard)| Temporary display and peripherals| For a one-time setup before we can control the Pi from our computer via SSH) | 5 - $10 |
| **Wiring Troubleshooting** | RPI Zero 2 Header & Jumper Wires | Temporary tools for troubleshooting wiring problems | For a one-time setup before we can solder the wire directly | 5 - $10 |
---


## 🛠️ Setting up the Pi

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

5. On startup, sometimes Chromium will have pop ups that can't be removed without a mouse. To bypass "restore tabs" pop up on reboots, type the following line by line:

```
//sudo nano is a command used to access/edit a file of a declared directory
sudo nano /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh

// add these under #!/bin/dash
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences

```

6. For launching in a clean fashion (no scroll bar, autoplay, no cursor, max screen visibility) we need to also edit this line in /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh

```
sudo nano /var/lib/dietpi/dietpi-software/installed/chromium-autostart.sh

// find the CHROMIUM_OPTS line of code and change accordingly
// note the "--window-size" and "--window-position" code will likely vary if you are using a display other than the 2.4" ILI9341
CHROMIUM_OPTS="--nocursor --no-memcheck--noerrdialogs --start-fullscreen --hide-scrollbars --autoplay-policy=no-user-gesture-required --start-maximized --force-device-scale-factor=.75 --high-dpi-support=1 --device-scale-factor=.75 --window-size=${RES_X:-320},${RES_Y:-240} --window-position=-35,40"

```

7. To display properly, we also need to change the Chromium resolution in /boot/dietpi.txt:

```
sudo nano /boot/dietpi.txt
//find the line reffering to the chromium resolutions and change according to your display ex: {RES_X:-320},${RES_Y:-240}

```

8. Now to get the actual video playback to fullscreen, we are going to use an extension called Enhancer for YouTube™ and enable the following settings:
- Extension Link: https://chromewebstore.google.com/detail/enhancer-for-youtube/ponfpcnoihfmfllpaingbgckeeldkhle?hl=en-US&utm_source=ext_sidebar
- Hide Shorts
- Hide related videos
- Hide chat
- Hide comments
- Automatically hide info cards and end screens
- Automatically expand the video player
- Use the available space based on the viewport dimensions to expand the video player

9. To get rid of pink video playback, disable graphic acceleration in chromium settings.

With this, you can now have your favorite be automatically ran on your monitor! Next, we will setup the SPI display for a more low-profile setup :)

## 🛠️ Setting up the ILI9341 Display

1. We will need to wire the ILI9341 to the RPI, following the diagram accordingly. (You can use jumper cables if you are not comfortable with soldering!)

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

1. Download and 3D Print 'Front Shell' and 'Back Shell' Files:
Download the [3D Assembly STEP File](cad/twice_monitor_final.step)

2. Use M2 nuts and bolts to assemble the chassis
