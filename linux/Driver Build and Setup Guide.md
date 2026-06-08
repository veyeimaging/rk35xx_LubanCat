# VEYE Camera SDK Integration Guide

## 1. SDK Overview

Two SDK packages are officially provided:

* **LubanCat_Linux_Generic_Full_SDK**
* **LubanCat_Linux_rk3588_SDK**

The SDK used for this integration is:

[LubanCat_Linux_Generic_Full_SDK](https://pan.baidu.com/s/19t8AZV9SYTdjn2uObBiSGA#list/path=%2Fsharelink1463230356-821055607364575%2F%E9%B2%81%E7%8F%AD%E7%8C%AB_%E7%91%9E%E8%8A%AF%E5%BE%AE%E7%B3%BB%E5%88%97%2F8-SDK%E6%BA%90%E7%A0%81%E5%8E%8B%E7%BC%A9%E5%8C%85%E5%8F%8A%E8%99%9A%E6%8B%9F%E6%9C%BA%2FLubanCat_Linux_SDK&parentPath=%2Fsharelink1463230356-821055607364575)


---

# 2. Driver Porting Procedure

## Step 1: Copy Camera Driver Sources

Copy all files from the **driver** directory into the kernel source tree:

```text
kernel-6.1/drivers/media/i2c
```

---

## Step 2: Copy Device Tree Overlay Files

Copy all files from the **overlay** directory into:

```text
kernel-6.1/arch/arm64/boot/dts/rockchip/overlay
```

Copy the following file:

```text
rk3588-lubancat-5io-csi.dtsi
```

to:

```text
kernel-6.1/arch/arm64/boot/dts/rockchip
```

---

## Step 3: Copy uEnv Configuration Files

Copy all files from the **uEnv** directory into:

```text
kernel-6.1/arch/arm64/boot/dts/rockchip/uEnv/rk3588
```

---

## Step 4: Configure the Kernel

From the SDK root directory, run:

```bash
./build.sh kconfig
```

Navigate to:

```text
Device Drivers
 └── Multimedia support
      └── Media ancillary drivers
           └── Camera sensor devices
```

Enable the following drivers:

```text
Camera sensor devices
[*] VEYE MV series camera support
[*] VEYE 2M series camera support
[*] VEYE GX series camera support
```

---

## Step 5: Build the Kernel Debian Package

Execute:

```bash
./build.sh kerneldeb
```

---

## Step 6: Install the New Kernel

Install the generated kernel package:

```bash
sudo dpkg -i linux-headers-6.1.99-rk3588_6.1.99-rk3588-23_arm64.deb
sudo reboot
```

---

# 3. Camera Enablement via Device Tree Overlay

Camera modules can be dynamically enabled on any CAM interface through the device tree overlay configuration.

### Important Notes

* Only one camera type can be enabled on each CAM interface.
* A system reboot is required after modifying the configuration.

Configuration file:

```text
/boot/uEnv/uEnv.txt
```

### Example: Enable MVCAM on CAM0

```text
# cam0
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-imx415-3840x2160-30fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-imx415-1920x1080-60fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-ov8858-3264x2448-15fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-ov8858-1632x1224-30fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-gc08a8-3264x2448-30fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-gc2053-1920x1080-30fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-gc4653-2560x1440-30fps-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-gxcam-overlay.dtbo
dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-mvcam-overlay.dtbo
#dtoverlay=/dtb/overlay/rk3588-lubancat-5io-cam0-veyecam2m-overlay.dtbo
```

---

# 4. Device Node Mapping

When all six CAM interfaces are enabled simultaneously, the device node assignments are as follows:

| CAM Interface | Media Device | Video Device |
| ------------- | ------------ | ------------ |
| CAM0          | /dev/media0  | /dev/video0  |
| CAM1          | /dev/media1  | /dev/video11 |
| CAM2          | /dev/media2  | /dev/video22 |
| CAM3          | /dev/media3  | /dev/video33 |
| CAM4          | /dev/media4  | /dev/video44 |
| CAM5          | /dev/media5  | /dev/video55 |

### Note

When only 1–5 camera interfaces are enabled, the mapping between camera modules and device nodes is not guaranteed to remain fixed.

Use the following command to identify the camera associated with a specific media device:

```bash
media-ctl -p -d /dev/media1 | head -n 20 ; media-ctl -p -d /dev/media1 | tail -n 20
```

Since the output can be lengthy, displaying only the first and last 20 lines is usually sufficient for quickly determining the device relationship.

---

# 5. Identifying the Camera Interface

Example output:

```text
...
device node name /dev/video11
...

<- "m00_b_gxcam 2-003b":0 [ENABLED]
...

entity 63: m00_b_gxcam 2-003b
...
```

From this output:

* `/dev/media1` corresponds to `/dev/video11`
* The sensor device is identified as `m00_b_gxcam 2-003b`
* The number `2` in the device name indicates the I²C bus number used by the camera

The I²C bus assignment for each CAM interface is shown below:

| CAM Interface | I²C Bus |
| ------------- | ------- |
| CAM0          | i2c0    |
| CAM1          | i2c1    |
| CAM2          | i2c2    |
| CAM3          | i2c5    |
| CAM4          | i2c7    |
| CAM5          | i2c8    |

Therefore:

```text
m00_b_gxcam 2-003b
```

indicates that the GXCAM module is connected to **CAM2**, since it is operating on **I²C bus 2**.

This method can be used to determine the corresponding physical camera interface for any detected camera device.
