# usbip-2024_setup
## Client-side (Windows)
Code and set-up instructions to run usbip client on Windows. Tested on Windows 11 (23H2) with usbip-win (0.3.6-dev).

1. Download the latest release of the USB over IP for windows package (usbip-win) from https://github.com/cezanne/usbip-win/releases.

    Note (2024-07-06): There is a **very promising and actively maintained** project called [usbip-win2](https://github.com/vadimgrn/usbip-win2) with GUI, but at the moment the drivers are unsigned and too unstable to be used for production .

2. Decompress the drivers and binaries to your destination folder (We will use `C:\usbip`)

3. Open PowerShell/Cmd as Admin and enter to the destination folder:
 and install the drivers:
    ```powershell
    cd C:\usbip
   ```

4. Install the drivers
    ```powershell
    pnputil /add-driver .\*.inf /install
    ```

### Manual testing

5. Check the available devices in the server:

    ```powershell
    ./usbip list -r <server_address>
    ```

    You should get a reply similar to:
    ```console
    Exportable USB devices
    ======================
    - raspi-usbip
        1-1.5: Brother Industries, Ltd : PT-P700 P-touch Label Printer (04f9:2061)
            : /sys/devices/platform/soc/3f980000.usb/usb1/1-1/1-1.5
            : (Defined at Interface level) (00/00/00)
    ```
    Where raspi-usbip is the <server_address> and 1-1.5 is the busID needed to attach the device.

6. Attach the device to the computer:
    ```powershell
    ./usbip attach -r <server_address> -b <busID>
    ```
    If the drivers were properly loaded and everything worked well you should get a message like this:
    ```console
    succesfully attached to port 0
    ```
    At this point the attached USB-over-IP device should appear normally like any other USB device in the Device Manager.

7. Detach an attached device:
    - First check the port of the desired device:
        ```powershell
        ./usbip port
        ```
        You should get something like:
        ```console
        Port 00: <Device in Use> at High Speed(480Mbps)
            Brother Industries, Ltd : PT-P700 P-touch Label Printer (04f9:2061)
            ?-? -> unknown host, remote port and remote busid
                -> remote bus/dev ???/???
        ```
    - Then detach the device:
        ```powershell
        ./usbip detach -p <port>
        ```

### Automating the procedure

The idea is that once the device is selected, with a simple click you can attach or detach the device to the computer in a trouble-free way.

You may need to run as Admin in PowerShell the following command to allow the execution of scripts:
```powershell
Set-ExecutionPolicy Unrestricted
```

1. Create a script to generate attaching and detaching mechanisms to a specific device in `C:\usbip\install_device.ps1`

```powershell
# Set the variable that will contain the list of devices available 
$devices = $null
# Keep asking for a valid server if $devices comes back empty
while ($devices -eq $null)
{
    # User input for the name or IP for the connection to the server
    $server_addr = Read-Host -Prompt "Please type the IP or the name of the server"
    # Request using usbip the list of available devices to attach
    # Run a RegEx pattern to recognize the BusID, VendorID and DeviceID and add the
    # matches to the list $devices.
    $devices = (./usbip list -r $server_addr | select-string -Pattern ".*([0-9]+-[0-9]+\.{0,1}[0-9]*).*\(([0-9a-zA-Z]{0,4}):([0-9a-zA-Z]{0,4})\)").Matches
}
echo "Devices available to attach:"
# Define the index counter to enumerate the list entries
$n=0 
# Loop through all the list entries and print them prefixed with their index
$devices | ForEach-Object -Process {echo "  ($n) $($_.Groups[0].Value)"; $n++}
# Define the variable that will contain the index of the selected device
$device_nr = -1
# Keep asking for a valid device index. That should be a valid int between 0 and $n-1
while ($device_nr -lt 0 -or $device_nr -ge $n)
{
    # User input for the index of the device
    [int]$device_nr = Read-Host -Prompt "Please select the number of the device to attach"
}

echo "Selected device: $devices[$device_nr].Groups[0].Value"
echo "Creating attachment script for VEN_$($devices[$device_nr].Groups[2].Value)&DEV_$($devices[$device_nr].Groups[3].Value)"
echo "As long as this device is available in the server, even if it changes port, the scripts will work"
echo "Searching for the BusID of the device"
$VEN = $devices[$device_nr].Groups[2].Value
$DEV = $devices[$device_nr].Groups[3].Value
$devices = (./usbip list -r $server_addr | select-string -Pattern ".*([0-9]+-[0-9]+\.{0,1}[0-9]*).*\($($VEN)\:$($DEV)\)").Matches
if($devices -ne $null)
{
    echo "Device found at BusID=$($devices.Groups[1].Value)"
}else{
	echo "Device not found"
}
```


```powershell

(./usbip list -r raspi-usbip | select-string -Pattern "([0-9]+-[0-9]+\.[0-9]+)(?=:).*(?<=\()([0-9a-zA-Z]{0,4}:[0-9a-zA-Z]{0,4})(?=\))").Matches.Groups[2].Value
```

```powershell
(Get-WmiObject Win32_PnPSignedDriver| where-object {$_.Description -Match "usbip-win VHCI Root" -or $_.DeviceName -Match "usbip-win VHCI Root"}) -eq $null
```
