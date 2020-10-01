# Resonance Testing
## The accelerometer
Currently all of this assumes that you are using the ADXL345 accelerometer. I think any breakout
board that you can find would suffice, but note that you may need to alter the wiring a little depending
on the breakout board pinout.
## Wiring
Just follow the guide (here)[https://github.com/nagimov/adxl345spi#wiring]
- Always ensure that you Pi is turned off when connecting and disconnecting accessories.
## Mounting
I think a lot of people are using this pinned STL file and mounting the board where the cable chain connects
to the extruder motor.

Personally, I just made a small mount for the breakout board and use some high strength double sided
kapton tape to mount the accelerometer anywhere on the toolhead.
## Getting the Code
This assumes moderate familiarity with git.
1. Check if you already have the proper remote available.
```bash
$ cd klipper
$ git remote -v
```
In that output if you see `https://github.com/dmbutyugin/klipper.git` as one of your remotes, then you have the
proper remote available, and can skip step 2.
2. Add Dmitry's fork as a remote source
```bash
$ cd ~/klipper
$ git remote add adxl https://github.com/dmbutyugin/klipper.git
```
confirm that it was successful with `$ git remote -v`
3. Checkout the latest code in the accelerometer branch and restart the service
```bash
$ git fetch adxl
$ git checkout --track adxl/adxl345-spi
$ sudo service klipper restart
```
4. Setup your RPi as an mcu if you haven't already. https://github.com/dmbutyugin/klipper/blob/adxl345-spi/docs/RPi_microcontroller.md
```bash
$ cd ~/klipper
$ sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu
$ sudo update-rc.d klipper_mcu defaults
$ make menuconfig
```
- in `make menuconfig` you will need to update the micro architecture to be `linux process`
- use the down arrow to select the `micro-controller architecture` line. Use right/left arrow keys
to highlight `select` at bottom and press `enter`
- use up/down arrow keys to select `linux process` it may not be visible until scrolling down
- once highlighted use right/left arrow keys to highlight `select` and press `enter`
- then highlight `exit` and press `enter`
5. Build and flash the code
```bash
$ sudo service klipper stop
$ make flash
$ sudo service klipper start
```
- Note: now you will have 2 services running on your system. `klipper` and `klipper_mcu`.
`klipper_mcu` needs to be running before `klipper` but your system should take care of that.
- if you need to stop/start/restart/check you can do that easily.
```bash
$ sudo service klipper_mcu restart
$ sudo service klipper restart
```
- replace `restart` with `stop`, `start` or `status` depending on your intent.
Additionally, you should have a link to a new psuedo terminal at `/tmp/klipper_host_mcu` (don't worry about what that is - just
know that is how you klipper will communicate the Pi behaving as MCU)
```bash
$ ls /tmp/klipper*
```
6. Ensure that you have the SPI bus enabled on your RPi
```bash
$ raspi-config
```
That will open a menu system where you need to scroll down to `Interfacing Options` and select it
then go down to `SPI` and click `enter`. Then enable SPI interface.
- Your RPi should reboot.
7. Update your config (printer.cfg)
```yaml
[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None
axes_map: x,y,z

[resonance_tester]
accel_chip: adxl345
probe_points:
    150, 150, 20
```
Depending on the orientation of your accelerometer you may need to adjust your `axes_map`.
Helping out another user, he had to adjust his to `axes_map: x,z,-y` essentially you need to
update your `axes_map` to match your accelerometer orientation. The first in the string aligns
with `X` axis, then `Y` axis and then `Z`. Consult your accelerometer breakout board datasheet to know 
which board axis matches the accelerometer axis and match those you your printer.
8. In octoprint or however else you send g-code to your printer execute the command `FIRMWARE_RESTART`
- Note on the PI you can do this:
```bash
$ echo firmware_restart > /tmp/printer
```
9. You should be all set to execute the commands `ACCELEROMETER_QUERY`, `MEASURE_AXES_NOISE` and the rest.

