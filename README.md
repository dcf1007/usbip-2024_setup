# usbip-2024_setup
Code modifications and set-up instructions to run usbip server on Debian12-based systems and the client in windows-based and linux-based systems

1. Install the USB over IP package:

    ```console
    apt install usbip
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

For the server-side 

 - https://www.unifix.org/2023/11/28/usbip-on-debian-12-usb-device-sharing-over-ip-network/
 - https://www.unifix.org/2023/11/28/automating-usbip-server/
 - https://wiki.archlinux.org/title/USB/IP
 - https://www.linux-magazine.com/Issues/2018/208/Tutorial-USB-IP
 - https://github.com/psct/usbip