# Board, connections and firmware

[← Start here](index.md) · [Next: First start →](first-start.md)

## What you need

| Item | What matters |
|---|---|
| ESP32-S3 board | Reference hardware: **YD-ESP32-S3 V1.3**, 16 MB flash, with two USB-C sockets. Firmware is configured for 16 MB flash. Do not substitute an ESP32, ESP32-C3 or arbitrary S3 board without checking compatibility. |
| Computer for installation | Windows, macOS or Linux with a desktop browser supporting Web Serial, such as Chrome or Edge. Subsequent DAC operation also works from a phone. |
| USB data cable to the computer | Connects to the board's **UART/COM socket**. A charge-only cable is insufficient. |
| USB-C-to-USB-B data cable to the DAC | USB-C connects to the board's **native USB socket**, USB-B to the DAC. Do not place a USB hub in this connection. |
| Stable USB power | Powers the Bridge through its UART socket; the computer can supply it initially. The DAC keeps its own power supply. |
| Wi-Fi | A reachable, password-protected **2.4 GHz** network; the phone/computer must be able to reach the Bridge on the same local network. |
| RMEBridge web installer | The dedicated browser installer for the RME Bridge, not the RoonPilot or IR Bridge installer. It supplies the matching firmware. |
| Optional IR transmitter and wires | Only needed for power on/off from the website. Its supply and signal input must suit the actual module and 3.3 V logic. No IR module is needed for USB-MIDI settings. |

**Flash** is the board's non-volatile storage. **Flashing** means writing the RME Bridge software into that storage. The software is called **firmware**. The planned web installer handles the writing for you. You do not need to program anything, select individual firmware files or enter memory addresses.

Place the exposed board on a dry, non-conductive surface. Keep screws, metal enclosures and loose wire ends away from contacts. Change GPIO wiring only with the Bridge switched off and disconnected from USB.

## Do not confuse the two USB sockets

![Schematic front view: antenna at the top, native USB at bottom left and UART at bottom right.](../assets/diagrams/board-en.svg)

The illustration shows the reference board with its antenna up and components facing you. Another revision may have different sockets, labels or power paths. Always check your board's labels as well.

| Socket | Purpose | Connect to |
|---|---|---|
| **UART / COM** | Flashing, serial boot messages and Bridge power | Computer during installation; USB power supply afterwards |
| **USB / native USB / OTG** | USB host for the RME DAC's MIDI connection | RME DAC using a USB-C-to-USB-B data cable |

A **USB host** controls a USB connection. In normal operation, the Bridge is the host and the DAC is the USB device. The native socket is therefore not a simultaneous second computer connection. Use the UART/COM socket for flashing and initially leave the DAC disconnected.

On the reference YD-ESP32-S3 V1.3 setup, the rear **USB-OTG solder bridge remains open**. No soldering is required for this setup. Do not close a jumper because instructions for a different board revision say to do so. A “USB-OTG” label does not automatically mean a jumper must be closed for this firmware.

## Prepare the computer

1. Connect only the UART/COM socket directly to the computer using a USB data cable. Temporarily disconnect other ESP boards where possible to avoid selecting the wrong device.
2. In the browser dialog, choose this board's **CH343/USB-to-serial port**. There is no fixed COM number. If identification is unclear, optionally use **Device Manager → Ports (COM & LPT)** on Windows to see which port appears when connecting it. On macOS, UART may appear as `cu.usbserial…` or a vendor-specific serial device.
3. If no port appears or an unknown device is shown, check the cable and socket first. The reference board uses a CH343 USB-to-serial chip. If a driver is needed, use the [official WCH CH343 download](https://www.wch-ic.com/downloads/CH343SER_EXE.html), not an arbitrary driver website. A differently populated board needs the driver for its actual chip.
4. Close serial monitors and other flashing programs. Normally only one program can use a serial port at a time.
5. Use a normal browser window and grant access to the serial port you selected when prompted. If Web Serial is unavailable, use a supported desktop browser. The phone is for subsequent operation, not a flashing requirement.

**Already installed RoonPilot or the IR Bridge?** The familiar sequence of data cable, browser device selection, writing, verification and restart remains the model. The port rule differs here: native “USB JTAG/serial debug unit” is not the UART connection used in this guide. Do not rotate the RME Bridge's plug 180 degrees to select another processor; it has two separate sockets and one ESP32-S3 for this procedure. Do not install RoonPilot or IR Bridge firmware on the RME Bridge.

## Installing the firmware: Flashing

Flashing is planned through a **dedicated RMEBridge web installer**: the same straightforward browser workflow as RoonPilot and the IR Bridge, using the matching RMEBridge firmware and the UART connection described here.

**The RMEBridge web installer will be provided separately and is not yet available.** This guide already describes the intended workflow to follow once it is supplied. Do not use the RoonPilot or IR Bridge installer instead.

### 1. Open the RMEBridge web installer

1. Once it is available, open the RMEBridge installation page in **Chrome or Edge on your computer**. The DAC control website on the Bridge and the web installer are different pages: the installer writes the software to the board; the control website is used afterwards.
2. Check that the page explicitly identifies **RME Bridge** and the matching ESP32-S3 board. The installer supplies the firmware; you do not need to download, extract or assign files.
3. Read the installation notes and acknowledge the prerequisites shown there. For an already configured Bridge, export profiles first and keep your Wi-Fi credentials and setup password.

### 2. Enter flashing mode

1. Leave the DAC disconnected. The board remains connected to the computer through its **UART/COM socket**.
2. Start the device connection in the web installer. The browser opens a list of serial ports. Choose the RME Bridge's previously identified **CH343/USB-to-serial port** and confirm. Its COM number may differ between computers.
3. The installer connects to the board and checks the detected chip. Continue only if the board matches the offered firmware.
4. If automatic connection fails: hold **BOOT**, briefly press and release **RESET**, then release **BOOT**. Try connecting again. If necessary, hold BOOT during connection and release it once the chip is identified. BOOT is not a setting to keep permanently enabled.
5. If no serial port appears, check the data cable, UART socket and, if necessary, the CH343 driver. If the browser reports a busy port, close other programs or browser tabs using the board.

### 3. Install and wait for completion

1. Select installation of the RMEBridge firmware in the installer. Read the notice for the displayed installation type before confirming.
2. **An installation with a full erase removes all Bridge settings, profiles and the setup password.** A new board needs its first setup. An already used board requires your saved information after an erase.
3. Keep the browser open and the USB cable connected throughout. The installer handles the required steps: erasing if applicable, writing the firmware and verifying the transferred data. You do not need to enter memory addresses or flash settings.
4. Wait for the final success message. A single progress bar at 100% may represent only one stage; do not disconnect the board before the entire installation finishes.
5. The Bridge restarts after installation. If it does not start automatically, briefly press **RESET** without holding BOOT. Keep it connected to the computer to read its setup password.

If installation stops, read the error message, check the connection and power supply, and retry with the RMEBridge web installer. A connection error is not a reason to use another project's firmware.

### 4. Read the setup password

On first start, the Bridge generates its own random **16-character password** for its setup Wi-Fi. There is no shared default password.

1. Finish the installation process first. Then open the web installer's **serial boot messages / Logs & Console**. This area shows text messages from the board, not the control website's Event log.
2. If the browser asks you to choose a device again, select the same UART/COM port. The console uses serial boot messages at 115200 baud; Wi-Fi speed and DAC settings are unrelated.
3. Briefly press the board's **RESET** button. Boot messages appear in the console.
4. Find `First setup: connect to RME-Bridge-… with password …`. Record the Wi-Fi name and password **privately**. This line contains the real password; do not put it in public screenshots or support posts.
5. Close or disconnect the console afterwards. The board can remain connected to the computer until Wi-Fi setup is complete.

If the Bridge already has Wi-Fi credentials, the first-start line may not appear again. The password is stored on the Bridge but cannot be retrieved from the Network status page. Keep it from the first start. [What to do if you lose it](troubleshooting.md#the-setup-password-is-lost).

## Connect an optional IR transmitter

USB-MIDI works without this step. IR is only needed to turn the DAC on or put it into standby from the website. The firmware drives **GPIO14**, not GPIO4. Model-specific power codes are built in; there is no learning procedure.

![Signal on GPIO14, a shared ground and a supply explicitly suited to the transmitter. The 5V pin is not used as a power output.](../assets/diagrams/ir-wiring-en.svg)

| Connection | Reference board connection | Important |
|---|---|---|
| Control signal | Pin labelled **14**, bottom left with antenna up | 3.3 V logic signal, not a power supply |
| Ground / GND | Pin labelled **G** / GND | Connect to the transmitter's ground |
| Module supply | **3V3 only for a module specified for it** | Check the actual module's supply voltage and current requirements |
| **5V / 5Vin** pin | Do not assume it supplies transmitter power | Not a guaranteed 5 V output in this setup |

Signal, positive supply and ground are arranged differently on different modules. Labels such as `S`, `+`, `−`, or wire colours do not establish a universal pinout. Follow your transmitter's datasheet. A 5 V module that appears to work at 3.3 V is not automatically specified for that supply.

If the module needs 5 V, use an appropriate, correctly designed supply. Do not simply connect another supply to `5Vin` or USB: this can back-feed a computer or power supply. Do not apply a 5 V signal to GPIO14. Use a transmitter with a suitable driver stage; never drive a high-power IR LED directly from the GPIO.

Aim the transmitter at the DAC's IR receiver. Keep metal and cable bundles away from the ESP32 antenna. Wires from the lower pins can run down the side. A short USB plug housing makes enclosure assembly easier.

## Updating versus starting over

A firmware update is not the same as a factory reset. Follow the RMEBridge web installer's notes for the version and installation type offered. An update **without a full erase** is not intended to remove saved settings; nevertheless, do not assume settings survive every firmware change. Export profiles first and retain the Wi-Fi details and setup password.

The current Bridge website has neither OTA upload nor a factory-reset button. An intentional fresh setup uses a complete installation with erasing in the planned web installer, once available. The next first start creates a new AP password; profiles are lost without a previous backup. Do not use this as the first remedy for a missing DAC response.

[Next: Set up Wi-Fi and connect the DAC →](first-start.md)
