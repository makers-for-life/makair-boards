# Raspberry Pi RTC

## Versions

| Version | Last Changelog | Ready? |
| ------- | -------------- | ------ |
| V1 | Initial design | ✅

## Notes

This RTC HAT was designed to avoid any conflict between Raspberry Pi touchscreen and RTC. All the RTC you can find are connected on SDA3/SCL3 of the Raspberry 4. But the touchscreen is already connected to this port, and there is some random issues.

So, this small board is designed to connect a DS3231 on I2C6 of the Raspberry 4 (pin 15 and 16).

### How to setup RTC on I2C6 of Raspberry 4

Make sure that the Raspberry Pi is up to date:

```
sudo apt update
sudo apt upgrade
sudo reboot
sudo rpi-update
```

* Append to `/boot/config.txt`:

`dtoverlay=i2c6`

* Make a new DTC from the existing RTC one (named `i2c6-rtc`);

`sudo dtc -I dtb -O dts /boot/overlays/i2c-rtc.dtbo -o /boot/overlays/i2c6-rtc.dts`

* Edit it to change `i2c_arm` to `i2c6`:

`sudo sed -i "s/i2c_arm/i2c6/g" /boot/overlays/i2c6-rtc.dts`

* Recompile it:

`sudo dtc -I dts -O dtb /boot/overlays/i2c6-rtc.dts -o /boot/overlays/i2c6-rtc.dtbo`

* Append to `/boot/config.txt`:

`dtoverlay=i2c6-rtc,ds3231`

After reboot, check `dmesg`:

```
dmesg | grep rtc
[    5.883603] rtc : registered as rtc0
```

If battery is low:

```
[  42.117539] rtc: low voltage detected, time is unreliable
```

Now write current time into the RTC, make sure your Raspberry Pi clock is synchronized, using `date`, then `sudo hwclock -w` to write current date into the RTC.

On Arch Linux, you have to create a script on startup to read RTC time during the boot:

```
cat > /etc/systemd/system/rtc.service << ENDRTCSERVICE
[Unit]
Description=RTClock
Before=network.target

[Service]
ExecStart=/usr/bin/hwclock -s
Type=oneshot
[Install]
WantedBy=multi-user.target
ENDRTCSERVICE
```

Then enable this service:

```
systemctl enable rtc
systemctl start rtc
```

Reboot, then check everything is OK:

```
journalctl -u rtc
```
