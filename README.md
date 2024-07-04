# usbip-2024_setup
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
3. Reload modules
    ```console
    sudo systemctl restart systemd-modules-load.service
    ```
    We can check that the modules loaded properly by running `lsmod`

Now you can list all the possible devices by running `usbip list -l` followed by `usbip bind --busid=bus_id` where bus_id has to be replaced by its kernel ID shown in the list.

What we wantt to do though is binding the devices in a plug'n'play manner. This can be by either whitelisting the device IDs to be exposed, or blacklisting the device IDs that should not be exposed.

4. Select which devices will be exported.

    





For the server-side 

 - https://www.unifix.org/2023/11/28/usbip-on-debian-12-usb-device-sharing-over-ip-network/
 - https://www.unifix.org/2023/11/28/automating-usbip-server/
 - https://wiki.archlinux.org/title/USB/IP
 - https://www.linux-magazine.com/Issues/2018/208/Tutorial-USB-IP
 - https://github.com/psct/usbip