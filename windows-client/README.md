# usbip-2024_setup
## Client-side (Windows)
Code and set-up instructions to run usbip client on Windows. Tested on Windows 11 (23H2) with usbip-win (0.3.6-dev).

### Manual testing

1. Download the latest release of the USB over IP for windows package (usbip-win) from https://github.com/cezanne/usbip-win/releases.

    Note (2024-07-06): There is a **very promising and actively maintained** project called [usbip-win2](https://github.com/vadimgrn/usbip-win2) with GUI, but at the moment the drivers are unsigned and too unstable to be used for production .

2. Decompress the drivers and binaries to your destination folder (We will use `C:\usbip`)

3. Open PowerShell as Admin and enter the destination folder:
 and install the drivers:
    ```powershell
    cd C:\usbip
   ```

4. Install the drivers
    ```powershell
     pnputil /add-driver .\*.inf /install
   ```

5. Check the available devices in the server:

    Note: We will use raspi-usbip as server address. You can use a domain name, an ipv4 or an ipv6.
    ```powershell
    .\usbip.exe list -r raspi-usbip
    ```

```powershell
(./usbip list -r raspi-usbip | select-string -Pattern "([0-9]+-[0-9]+\.[0-9]+)(?=:).*(?<=\()([0-9a-zA-Z]{0,4}:[0-9a-zA-Z]{0,4})(?=\))").Matches.Groups[2].Value
```

```powershell
(Get-WmiObject Win32_PnPSignedDriver| where-object {$_.Description -Match "usbip-win VHCI Root" -or $_.DeviceName -Match "usbip-win VHCI Root"}) -eq $null
```
