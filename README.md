# Raspberry Pi LED Matrix NCAA Men's Hockey Scoreboard

Work in progress fork of the awesome [rpi-led-nhl-scoreboard](https://github.com/gidger/rpi-led-nhl-scoreboard) which was inspired by [nhl-led-scoreboard](https://github.com/riffnshred/nhl-led-scoreboard).

Display live NCAA Men's Ice Hockey game scores, start times, etc. on a LED matrix driven by a Raspberry Pi. Makes use of the unofficial NCAA API for all game information. Displays the current day's games unless it is before 10:00 AM local time, when it will display the previous night's outcomes.

![Example](https://github.com/byrneta/collegehockey-led-scoreboard/blob/main/examples/scoreboard.gif)

## To-Do
1. Add logic to display upcoming day's games if there are no games on a given day

## Notes
1. Division II/III/other exhibition opponents will appear as the NCAA logo

## Installation Instructions
These instructions assume some basic knowledge of Unix and how to edit files via the command line.
1. Flash an SD card with [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) on your personal computer.

2. Unplug and replug the SD card.

3. Add an empty file named "ssh" to the boot directory on the SD card. Navigate to the SD card and enter the following command.
    ```bash
    touch ssh
    ```

4. Add and configure wpa_supplicant.conf.

    ```
    touch wpa_supplicant.conf
    ```

    Add the following to wpa_supplicant.conf using your text editor of choice. Configure the network information and two digit [country code](https://www.iban.com/country-codes) as per your needs.
    ```
    country=US
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="NETWORK-NAME"
        psk="NETWORK-PASSWORD"
    }
    ```

5. Remove the SD card from your personal computer and insert it into your Raspberry Pi. Boot up and SSH into your RPi.

6. Set location/time zone and new password via [raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md).
    ```bash
    sudo raspi-config
    ```

7. Get the latest updates.
    ```bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    ```

8. Disable on-board sound.
    ```
    sudo nano /boot/config.txt
    ```
    Edit the dtparam line to look like this:
    ```
    dtparam=audio=off
    ```

9. Disable sleep. 

    ```bash
    sudo nano /etc/rc.local
    ```

    Above the line that says "exit 0" insert the following command and save the file:
    ```
    /sbin/iw wlan0 set power_save off
    ```

10. Install pip3.
    ```bash
    sudo apt-get install python3-pip -y
    ```

11. Install git.
    ```bash
    sudo apt-get install git -y
    ```

12. Copy this repository to your home directory.
    ```bash
    cd ~/

    git clone --recursive https://github.com/byrneta/collegehockey-led-scoreboard.git
    ```

13. Install the LED Matrix Python package. Navagate to the root directory of the matrix library (/submodules/rpi-rgb-led-matrix) and enter the following commands.
    ```bash
    cd ~/collegehockey-led-scoreboard/submodules/rpi-rgb-led-matrix

    sudo apt-get update && sudo apt-get install python3-dev python3-pillow -y

    make build-python PYTHON=$(which python3)

    sudo make install-python PYTHON=$(which python3)
    ```

14. Install any missing requirements. Return to the root of your clone of this repository and enter the following command.

    ```bash
    cd ~/collegehockey-led-scoreboard

    pip3 install -r requirements.txt
    ```

15. Make main code run at RPi startup.

    ```bash
    nano ~/start-scoreboard.sh
    ```
    Copy-paste the following:
    ```
    #!/bin/bash
    cd /home/pi/collegehockey-led-scoreboard
    n=0
    until [ $n -ge 10 ]
    do
       sudo python3 collegehockey-led-scoreboard.py  && break
       n=$[$n+1]
       sleep 10
    done
    ```

    **OPTIONAL**: you can make this repository automatically stay up to date by using this edited version of the above start-scoreboard.sh. This will check for updates on reboot before starting the program. I generally would not advise doing this, but it may be useful when making as a gift for a non tech savvy person. Note that this will prevent anything from being displayed if there's no internet connection (not that anything of interest would be displayed without internet anyhow...).
    ```
    #!/bin/bash
    cd /home/pi/collegehockey-led-scoreboard
    
    while ! ping -c 1 -W 1 github.com; do
        echo "Waiting for GitHub..."
        sleep 1
    done

    git pull origin main

    n=0
    until [ $n -ge 10 ]
    do
        sudo python3 collegehockey-led-scoreboard.py  && break
        n=$[$n+1]
        sleep 10
    done
    ```

    Save and exit. Now, let's make that script executable:

    ```
    chmod +x ~/start-scoreboard.sh
    ```

    Make the script run on boot:

    ```
    sudo crontab -e
    ```
    Add the following command to the bottom:

    ```
    @reboot /home/pi/start-scoreboard.sh > /home/pi/cron.log 2>&1
    ```

    Save and exit. Finally, test your change by rebooting your RPi.

    ```
    sudo reboot
    ```