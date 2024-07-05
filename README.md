# usbip-2024_setup
## Server-side (Linux -debian based-)
Code modifications and set-up instructions to run usbip server on Debian12-based systems and the client in windows-based and linux-based systems

1. Install the USB over IP package:

    ```console
    sudo apt install usbip
    ```
2. Add the required modules to `/etc/modules`
    ```console
    usbip_core
    usbip_host
    ```
3. Reload the modules
    ```console
    sudo systemctl restart systemd-modules-load.service
    ```
    We can check that the modules loaded properly by running `lsmod`

Now you can list all the possible devices by running `usbip list -l` followed by `usbip bind --busid=bus_id` where bus_id has to be replaced by its kernel ID shown in the list. Finally we could start the daemon by running `usbipd`

But what we want to do though is binding the devices in a plug'n'play fashon and execute the daemon in a persistent manner. This can be by either whitelisting the device IDs to be exposed, or blacklisting the device IDs that should not be exposed.

4. Create a new service for usbipd at `/etc/systemd/system/usbipd.service`:
    ```
    [Unit]
    Description=USB over IP daemon service
    After=network-online.target
    Wants=network-online.target

    [Service]
    # Type forking because it daemonizes
    Type=forking

    # If the process is killed, exited or terminated, restart it
    Restart=always

    # Start usbpipd as a daemon
    ExecStart=/usr/sbin/usbipd -D

    [Install]
    WantedBy=multi-user.target
    ```

5. Create an Event Handler for detecting plugged USB devices at `/etc/udev/rules.d/90-usbip.rules`:
    ```
    # Check if the event is triggered by a USB. If not skip the execution
    SUBSYSTEM!="usb", GOTO="usbip_exit"

    # ---------------------------- START DEVICE FILTERING MODE BLOCK ---------------------------- #
    
    # ¡¡¡ATTENTION!!!: ONLY ONE OF THE MODES SHOULD BE UNCOMMENTED!
    
    ##########################################
    #     Skip the following USB devices     #
    ##########################################
    #
    # Rpi integrated peripherials. Modify accordingly to your host hardware
    # SMSC9512/9514 Fast Ethernet Adapter
    #ATTR{idVendor}=="0424", ATTR{idProduct}=="ec00", GOTO="usbip_exit"
    #
    # Linux Foundation 2.0 root hub
    #ATTR{idVendor}=="1d6b", ATTR{idProduct}=="0002", GOTO="usbip_exit"
    #
    # Other USB devices to exclude:
    # Logitech, Inc. Nano Receiver
    #ATTR{idVendor}=="046d", ATTR{idProduct}=="c534", GOTO="usbip_exit"

    ##########################################
    # Include ONLY the following USB devices #
    ##########################################
    
    # Brother Industries, Ltd PT-P700 P-touch Label Printer
    ATTR{idVendor}=="04f9", ATTR{idProduct}=="2061", GOTO="usbip_bind"
    
    # Skip all other USB devices
    GOTO="usbip_exit"

    # ----------------------------- END DEVICE FILTERING MODE BLOCK ----------------------------- #

    # Trigger the export (IP binding) event for the USB device
    LABEL="usbip_bind"
    ACTION=="add|remove", ENV{DEVTYPE}=="usb_device", TAG+="usbip", TAG+="systemd", ENV{SYSTEMD_WANTS}+="usbip@%k.service"

    #End of the execution blocks
    LABEL="usbip_exit"

    ```




10. Reload and enable the usbipd.service

    ```
    sudo systemctl --system daemon-reload

    sudo systemctl enable usbipd.service

    sudo systemctl restart usbipd.service
    ```


Resources:
 - https://www.unifix.org/2023/11/28/usbip-on-debian-12-usb-device-sharing-over-ip-network/
 - https://www.unifix.org/2023/11/28/automating-usbip-server/
 - https://wiki.archlinux.org/title/USB/IP
 - https://www.linux-magazine.com/Issues/2018/208/Tutorial-USB-IP
 - https://github.com/psct/usbip
 - https://github.com/alpertsev/usbip-service-shell
