# Raspberry Pi LED Matrix NCAA Hockey Scoreboard

Work in progress fork of the awesome [rpi-led-nhl-scoreboard](https://github.com/gidger/rpi-led-nhl-scoreboard) which was inspired by [nhl-led-scoreboard](https://github.com/riffnshred/nhl-led-scoreboard).

Display live NCAA game scores, start times, etc. on a LED matrix driven by a Raspberry Pi. Makes use of the unofficial NCAA API for all game information.

## Installation Instructions
These instructitons assume some basic knowledge of Unix and how to edit files via the command line.
1. Flash an SD card with [Rasberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/) on your personal computer.

2. Unplug and replug the SD card.

3. Add an empty file named "ssh" to the boot directory on the SD card. Navagate to the SD card and enter the following command.
    ```bash
    touch ssh
    ```

4. Add and configure wpa_supplicant.conf.

    ```
    touch wpa_supplicant.conf
    ```

    Add the following to wpa_supplicant.conf using your text editor of choice. Configure the network information and two didgit [country code](https://www.iban.com/country-codes) as per your needs.
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

    Above the line that says exit 0 insert the following command and save the file:
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

12. Copy this repository.
    ```bash
    git clone --recursive https://github.com/gidger/rpi-led-nhl-scoreboard.git
    ```

13. Install the LED Matrix Python package. Navagate to the root directory of the matrix library (/submodules/rpi-rgb-led-matrix @ dfc27c1) and enter the following commands.
    ```bash
    sudo apt-get update && sudo apt-get install python3-dev python3-pillow -y

    make build-python PYTHON=$(which python3)

    sudo make install-python PYTHON=$(which python3)
    ```

14. Install any missing requirements. Return to the root of your clone of this repository and enter the following command.

    ```bash
    pip3 install -r requirements.txt
    ```

15. Make main code run at RPi startup.

    ```bash
    nano ~/start-scoreboard.sh
    ```
    Copy-paste the following:
    ```
    #!/bin/bash
    cd /home/pi/rpi-led-nhl-scoreboard
    n=0
    until [ $n -ge 10 ]
    do
       sudo python3 rpi-led-nhl-scoreboard.py  && break
       n=$[$n+1]
       sleep 10
    done
    ```

    Save and exit by pressing CTRL-X, then Y, and then Enter.

    Make that script executable:

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