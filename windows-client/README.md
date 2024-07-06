
```powershell
(./usbip list -r raspi-usbip | select-string -Pattern "([0-9]+-[0-9]+\.[0-9]+)(?=:).*(?<=\()([0-9a-zA-Z]{0,4}:[0-9a-zA-Z]{0,4})(?=\))").Matches.Groups[2].Value
```
