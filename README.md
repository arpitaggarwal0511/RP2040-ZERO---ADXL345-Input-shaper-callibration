# RP2040-Zero Klipper ADXL345 Input Shaper

Use a **Waveshare RP2040-Zero** as an external Klipper MCU for an **ADXL345 accelerometer**, allowing Klipper to perform resonance measurements and input-shaper calibration without using the printer's main controller.

## Hardware

* Waveshare RP2040-Zero
* ADXL345 accelerometer
* Klipper-based 3D printer
* USB connection between RP2040-Zero and the Klipper host

## 1. Get Klipper Source

Clone the Klipper repository and enter the directory:

```bash
git clone https://github.com/Klipper3d/klipper.git
cd klipper
```

Verify the repository:

```bash
git status
git describe --always --tags
```

Expected output will resemble:

```text
On branch master
nothing to commit, working tree clean

v0.xx.x
```

The exact version depends on the Klipper revision being used.

## 2. Configure Klipper for RP2040

Open the Klipper configuration menu:

```bash
make menuconfig
```

Set:

```text
Micro-controller Architecture: Raspberry Pi RP2040
Bootloader offset:             No bootloader
Communication interface:       USB
```

Then select:

```text
Save & Exit
```

## 3. Install the ARM Build Toolchain

Install the compiler required to build firmware for the RP2040:

```bash
sudo apt update
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi libnewlib-arm-none-eabi
```

Verify the compiler:

```bash
arm-none-eabi-gcc --version
```

Expected output should contain:

```text
arm-none-eabi-gcc ...
```

## 4. Build the Klipper Firmware

From the Klipper directory:

```bash
make
```

A successful build ends with:

```text
Building out/lib/rp2040/elf2uf2/elf2uf2
Creating uf2 file out/klipper.uf2
```

The resulting firmware is:

```text
out/klipper.uf2
```

## 5. Enter RP2040-Zero BOOTSEL Mode

1. Disconnect the RP2040-Zero from USB.
2. Hold the **BOOT** button.
3. Connect the USB cable.
4. Release the **BOOT** button.

Windows should detect a removable drive named:

```text
RPI-RP2
```

## 6. Flash Klipper Firmware

Copy:

```text
out/klipper.uf2
```

to the `RPI-RP2` drive.

### WSL

If the drive is not automatically available under `/mnt`, find its Windows drive letter:

```bash
powershell.exe -Command "Get-Volume | Where-Object FileSystemLabel -eq 'RPI-RP2' | Select-Object DriveLetter,FileSystemLabel"
```

Example:

```text
DriveLetter FileSystemLabel
----------- ---------------
F           RPI-RP2
```

Mount the drive in WSL:

```bash
sudo mkdir -p /mnt/rp2040
sudo mount -t drvfs F: /mnt/rp2040
```

Verify the bootloader drive:

```bash
ls -la /mnt/rp2040
```

Expected files include:

```text
INDEX.HTM
INFO_UF2.TXT
```

Copy the firmware:

```bash
cp ~/klipper/out/klipper.uf2 /mnt/rp2040/
```

The RP2040-Zero should automatically reboot after the firmware is copied.

The `RPI-RP2` drive will disappear. This is expected.

## 7. Verify Klipper Firmware

Reconnect the RP2040-Zero normally without holding the BOOT button.

On Windows, verify that the board appears as a USB serial device:

```powershell
Get-PnpDevice -PresentOnly | Where-Object { $_.Class -eq "Ports" } | Format-Table Status,FriendlyName,InstanceId -Auto
```

Expected output:

```text
Status FriendlyName             InstanceId
------ ------------             ----------
OK     USB Serial Device (COMx) USB\VID_1D50&PID_614E\...
```

The important values are:

```text
Status: OK
VID:    1D50
PID:    614E
```

This confirms that the RP2040-Zero has successfully booted into Klipper firmware.

## Current Status

At this point, the RP2040-Zero has:

* Klipper firmware configured for RP2040
* Klipper firmware successfully compiled
* Klipper firmware successfully flashed
* Successfully rebooted from BOOTSEL mode
* Successfully enumerated as a Klipper USB device

The next stage is connecting the **ADXL345 to the RP2040-Zero via SPI** and configuring the RP2040-Zero as a secondary Klipper MCU for resonance measurements and input-shaper calibration.
