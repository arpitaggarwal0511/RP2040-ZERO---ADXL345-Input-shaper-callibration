# RP2040-Zero Klipper ADXL345 Input Shaper

Waveshare RP2040-Zero as an external Klipper MCU for an ADXL345 accelerometer.

## Hardware

* Waveshare RP2040-Zero
* ADXL345
* Klipper 3D printer

## 1. Configure Klipper for RP2040

```bash
make menuconfig
```

Set:

```text
Micro-controller Architecture: Raspberry Pi RP2040
Bootloader offset:             No bootloader
Communication interface:       USB
```

Select **Save & Exit**.

## 2. Install Build Dependencies

```bash
sudo apt update
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi libnewlib-arm-none-eabi
```

Verify:

```bash
arm-none-eabi-gcc --version
```

## 3. Build Klipper Firmware

```bash
make
```

Successful build:

```text
Building out/lib/rp2040/elf2uf2/elf2uf2
Creating uf2 file out/klipper.uf2
```

Firmware:

```text
out/klipper.uf2
```

## 4. Enter BOOTSEL Mode

1. Disconnect the RP2040-Zero.
2. Hold **BOOT**.
3. Connect USB.
4. Release **BOOT**.

The board appears as:

```text
RPI-RP2
```

## 5. Flash Firmware

### Windows

Copy:

```text
out/klipper.uf2
```

to the `RPI-RP2` drive.

### WSL

Find the `RPI-RP2` drive:

```bash
powershell.exe -Command "Get-Volume | Where-Object FileSystemLabel -eq 'RPI-RP2' | Select-Object DriveLetter,FileSystemLabel"
```

Mount it:

```bash
sudo mkdir -p /mnt/rp2040
sudo mount -t drvfs F: /mnt/rp2040
```

Replace `F:` with the detected drive letter.

Verify:

```bash
ls -la /mnt/rp2040
```

Expected:

```text
INDEX.HTM
INFO_UF2.TXT
```

Flash:

```bash
cp out/klipper.uf2 /mnt/rp2040/
```

The `RPI-RP2` drive disappears after flashing.

## 6. Verify Klipper Firmware

Reconnect the RP2040-Zero normally without holding **BOOT**.

On Windows:

```powershell
Get-PnpDevice -PresentOnly | Where-Object { $_.Class -eq "Ports" } | Format-Table Status,FriendlyName,InstanceId -Auto
```

Expected:

```text
Status FriendlyName             InstanceId
------ ------------             ----------
OK     USB Serial Device (COMx) USB\VID_1D50&PID_614E\...
```

The RP2040-Zero is now running Klipper firmware and is ready to be connected to the printer's Klipper host as an external MCU.
