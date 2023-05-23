---
author: "Kalyan"
title: "Understanding iio devices in Linux"
date: "2023-05-23"
description: "Understanding iio devices in Linux"
tags: ["beaglebone", "c++", "cross-compile", "linux", "drivers", "device-drivers", "iio", "mpu9250"]
ShowToc: true
math: true
---
# Understaning iio devices in Linux
The Industrial I/O core offers a unified framework for writing drivers for many different types of Embedded sensors. a standard interface to user space applications manipulating sensors.
## IIO device sysfs interface
Attributes are sysfs files used to expose chip info and also allowing applications to set various configuration parameters. For device with index X, attributes can be found under `/sys/bus/iio/iio:deviceX/` directory
To look at all the attributes and their explanation, please take a look here
[linux/sysfs-bus-iio](https://github.com/torvalds/linux/blob/master/Documentation/ABI/testing/sysfs-bus-iio)

---
## iio driver MPU9250
The BeagleBone Blue is a popular development board that features an integrated MPU9250 sensor. The MPU9250 is a System-in-Package (SiP) that combines a 3-axis accelerometer, a 3-axis gyroscope, and a 3-axis magnetometer. It is commonly used for motion sensing and orientation detection in various applications.

On the BeagleBone Blue, the MPU9250 sensor is typically exposed as an IIO (Industrial I/O) device. The Industrial I/O (IIO) framework in the Linux kernel provides a unified interface for accessing various types of sensors and other industrial devices.

To interact with the MPU9250 sensor on the BeagleBone Blue, you can access the IIO device interface provided by the Linux kernel. This interface allows you to read sensor data, configure sensor settings, and perform other operations specific to the MPU9250 sensor. 

The exact path to the IIO device for the MPU9250 on the BeagleBone Blue may vary depending on the specific Linux distribution and kernel version you are using. However, you can typically find it under the `/sys/bus/iio/devices/` directory.

The Beaglebone blue's `/sys/bus/iio/devices` look like this.
```bash
debian@BeagleBone:/sys/bus/iio/devices$ ls
iio:device0  iio:device1  iio:device2  iio:device3  trigger0
```
As we can see there are multiple devices and the device number of MPU9250 may or maynot change after the restart. So one way we can access the device is going through all the devices in `/sys/bus/iio/device` and check the name of the device.
The current Beaglebone blue I am having has multiple devices.
*device0*
```bash
debian@BeagleBone:/sys/bus/iio/devices$ cat iio\:device0/name
TI-am335x-adc.0.auto
```
*device1*
```bash
debian@BeagleBone:/sys/bus/iio/devices$ cat iio\:device1/name
bmp280
```
*device2*
```bash
debian@BeagleBone:/sys/bus/iio/devices$ cat iio\:device2/name
mpu9250
```
MPU9250 is at device2 on this run. It might change to another number after a restart. 
*device3*
```bash
debian@BeagleBone:/sys/bus/iio/devices$ cat iio\:device3/name
ak8975
```
## Configuration Parameters of MPU9250
The configuration parameters of the MPU9250 has been exposed as a file system files. 
The configuration parameters can be found at
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ ls
buffer                    in_accel_y_calibbias        in_anglvel_y_calibbias  of_node
current_timestamp_clock   in_accel_y_raw              in_anglvel_y_raw        power
dev                       in_accel_z_calibbias        in_anglvel_z_calibbias  sampling_frequency
in_accel_matrix           in_accel_z_raw              in_anglvel_z_raw        sampling_frequency_available
in_accel_mount_matrix     in_anglvel_mount_matrix     in_gyro_matrix          scan_elements
in_accel_scale            in_anglvel_scale            in_temp_offset          subsystem
in_accel_scale_available  in_anglvel_scale_available  in_temp_raw             trigger
in_accel_x_calibbias      in_anglvel_x_calibbias      in_temp_scale           uevent
in_accel_x_raw            in_anglvel_x_raw            name
```
### buffer
The buffer directory contains interfaces for elements that will be captured for a single triggered sample set in the buffer.
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ ls buffer/
data_available  enable  length  watermark
```
#### data_available
Read only value indicating the number of bytes of data available in the buffer. In the case of an output buffer, this indicates the amount of empty space available to write data to. In the case of an input buffer, this indicates the amount of data available for reading.
#### enable
Actually start the buffer capture up. Will start the trigger of the device.
{{< notice info >}}
> It might not be possible to change the configuration parameters once the buffer has been enabled and the device started collecting data. i.e. If we want to change the sampling_frequency, we need to do it before the buffer is enabled. 
{{< /notice >}}

#### length
Length of the buffer.
#### watermark
Positive integer specifying the maximum number of scan elements to wait for.
*Poll will block until the watermark is reached*
In the context of the `/sys/bus/iio/devices/iio:device2/buffer/watermark` file, the term "watermark" refers to a threshold or level that determines when the buffer associated with the IIO device is considered full or empty.

The IIO buffer watermark is used to control how much data is stored in the buffer before triggering specific actions or events. It helps manage the buffer's capacity and allows for efficient data handling.

The `/sys/bus/iio/devices/iio:device2/buffer/watermark` file represents the watermark level of the buffer associated with `iio:device2`. It specifies the number of data samples that should be present in the buffer before triggering a specific action.

For example, if the watermark is set to a value of 100, it means that the buffer should contain 100 data samples before a particular event or action is triggered. This event could be an interrupt to notify the system that the buffer is full and needs to be processed or emptied.

By adjusting the watermark value, you can control the buffer's behavior and fine-tune it according to your application's requirements. 

{{< notice tip >}}
> Setting a higher watermark can reduce the frequency of buffer-related events, while setting a lower watermark can provide more frequent updates or notifications.
{{< /notice >}}

To modify the watermark value, you can write the desired value to the `/sys/bus/iio/devices/iio:device2/buffer/watermark` file using the appropriate file writing methods or utilities.
### current_timestamp_clock
The type of clock used to timestamp buffered samples and events for the device.
### in_accel_mount_matrix
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_mount_matrix
1, 0, 0; 0, 1, 0; 0, 0, 1
```
Mounting matrix for IIO sensors. This is a rotation matrix that informs userspace about sensor chip's placement relative to the main hardware it is mounted on.
### in_accel_scale
The `in_accel_scale` attribute allows you to read or set the scaling factor applied to the accelerometer's raw readings. This scaling factor is used to convert the raw sensor values into meaningful units of acceleration, such as meters per second squared (m/s²) or gravitational forces (g).
From the [driver implementation]([linux/inv_mpu_core.c at master · torvalds/linux · GitHub](https://github.com/torvalds/linux/blob/master/drivers/iio/imu/inv_mpu6050/inv_mpu_core.c)) we can see that the values are represented in m/s2 and if we multiply the raw values with the scale, we get values in m/s2 and rad/s
```cpp
/*
 * this is the gyro scale translated from dynamic range plus/minus
 * {250, 500, 1000, 2000} to rad/s
 */
static const int gyro_scale_6050[] = {133090, 266181, 532362, 1064724};

/*
 * this is the accel scale translated from dynamic range plus/minus
 * {2, 4, 8, 16} to m/s^2
 */
static const int accel_scale[] = {598, 1196, 2392, 4785};
```
The [[MPU9250]] has been configured in `2G` mode
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_scale
0.000598
```
### in_accel_scale_available
According to the data sheet, we have 4 *Full scale select* values.
![Accel Full Scale Select](accel_config.png)
If we select `ACCEL_FS_SEL = 2g(00)`, the the highest that the Accelerometer is going to measure is +2g to -2g. Correspondingly, the higher we go the higher the Accelerometer is going to measure but the sensitivity is going to decrease. 
Now lets see how the MPU9250 driver can be configured by looking at the available scales.
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_scale_available
0.000598 0.001196 0.002392 0.004785
```
#### How are these scales calculated
Inorder to figure out how they are calculated, we need to know what is *Sensitivity Scale Factor*. 
![Accelerometer Full Scale Range](accel_fs_sel.png)
The sensitivity scale factor of the MPU9250 refers to the conversion factor used to convert the raw sensor readings of the accelerometer or gyroscope into meaningful physical units. If I set *AFS_SEL = 0*, then the Sensitivity Scale Factor is going to become **16,384** according to the picture above. In other words, the accelerometer is going to give a value of **16,384** for 1g of acceleration experienced by it. Since we want the values in m/sec2, we need to multiply the values with 9.8 as $$1g = 9.8m/sec2$$
The scales are derived as follows:
$$
\begin{align}
scale 1 = \frac{2}{16384 * 2} * 9.8 = 0.000598 \\\\
scale 2 = \frac{4}{16384 * 2} * 9.8 = 0.001196 \\\\
scale 3 = \frac{8}{16384 * 2} * 9.8 = 0.002392 \\\\
scale 4 = \frac{16}{16384 * 2} * 9.8 = 0.00478
\end{align}
$$
We can also understand it as the scale to convert raw values that we get into g's

If we select *AFS_SEL = 1*, then we have to multiply the raw value with $\frac{1}{8192}$ to get the value in g. Then if we want to get the value in $m/sec2$ then we need to multiply $\frac{1}{8192} * 9.8$ and we get **0.001196** which is the second scale.
We need to write the scale we want in [in_accel_scale](#in_accel_scale) i.e. if we want to measure from -4g to +4g, then we need to write **0.001196** into [in_accel_scale](#in_accel_scale).
### in_accel_x(y_z)calibbias
The hardware applied calibration offset(assumed to fix production inaccuracies)

{{< notice warning >}}
> The calibration bias values seem to be strange. Its best not to use them.
```
> debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_x_calibbias
4852
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_y_calibbias
18
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_accel_z_calibbias
-28672
```
{{< /notice >}}

### in_accel_x(y_z)raw
Acceleration in x, y or z. Units after application of scale and offset are $m/sec2$.
### in_anglvel_scale
As explained in [in_accel_scale_available](#in_accel_scale_available) section, the Gyroscope also has scale calculated to convert raw measurements into $rad/sec$
### in_anglvel_scale_available
In the [datasheet]([PS-MPU-9250A-01-v1.1.pdf (tdk.com)](https://invensense.tdk.com/wp-content/uploads/2015/02/PS-MPU-9250A-01-v1.1.pdf)) we can see *Full-Scale Range* for Gyroscope just as [in_accel_scale_available](in_accel_scale_available) 
![Gyroscope Full-Scale Range](gyro_fs_sel.png)
If we select *FS_SEL = 0*, then for each $deg/sec$, the sensor will output 131 and it decreases as we want to measure more. If *FS_SEL = 0*, the highest value that the sensor is going to measure is in the range of +250 to -250$deg/sec$ 
There are 4 scales available that correspond to the *FS_SEL* values shown in the table above
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat in_anglvel_scale_available
0.000133090 0.000266181 0.000532362 0.001064724
```
They are caluculated as follows
$$
\begin{align}
Scale 1 = \frac{1}{131}°/sec = \frac{1}{131} * 0.017453 =  0.000133229\\\\
Scale 2 = \frac{1}{65.5} * 0.017453 = 0.000266458 rad/sec\\\\
Scale 3 = \frac{1}{32.8} * 0.017453 = 0.0005321
\end{align}
$$


### scan_elements
The **scan_elements** folder contains variety of important configuration parameters.  
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ ls -l scan_elements/
total 0
-rw-rw-r-- 1 root gpio 4096 May  6 06:51 in_accel_x_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_x_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_x_type
-rw-rw-r-- 1 root gpio 4096 May  6 06:53 in_accel_y_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_y_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_y_type
-rw-rw-r-- 1 root gpio 4096 May  6 06:53 in_accel_z_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_z_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_accel_z_type
-rw-rw-r-- 1 root gpio 4096 May  6 07:00 in_anglvel_x_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_x_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_x_type
-rw-rw-r-- 1 root gpio 4096 May  6 07:00 in_anglvel_y_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_y_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_y_type
-rw-rw-r-- 1 root gpio 4096 May  6 07:00 in_anglvel_z_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_z_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_anglvel_z_type
-rw-rw-r-- 1 root gpio 4096 May  6 06:36 in_temp_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_temp_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_temp_type
-rw-rw-r-- 1 root gpio 4096 May  6 06:36 in_timestamp_en
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_timestamp_index
-r--r--r-- 1 root gpio 4096 May  6 06:36 in_timestamp_type
```
#### in_accel_x(y_z)en
Writing 1 to this file enables the Accelerometer's x, y and z values to be written to the buffer.
#### in_accel_x(y_z)type
Description of the element data storage within the buffer and hence the form in which it is read from user-space.
The file will have contents of the form
```bash
[be|le]:[s|u]bits/storagebits[>>shift]
```
*be* or *le* signifies big or little Endian. *s* or *u* specifies if the element is signed or unsigned. bits is the number of bits
of data and storagebits is the space (after padding) that it occupies in the buffer.
From the [implementation]([linux/inv_mpu_core.c at v5.10 · torvalds/linux · GitHub](https://github.com/torvalds/linux/blob/v5.10/drivers/iio/imu/inv_mpu6050/inv_mpu_core.c#LL1037C1-L1054C3)) of the driver we can see it has been defined as Big Endian .
```c
#define INV_MPU6050_CHAN(_type, _channel2, _index)                    \
	{                                                             \
		.type = _type,                                        \
		.modified = 1,                                        \
		.channel2 = _channel2,                                \
		.info_mask_shared_by_type = BIT(IIO_CHAN_INFO_SCALE), \
		.info_mask_separate = BIT(IIO_CHAN_INFO_RAW) |	      \
				      BIT(IIO_CHAN_INFO_CALIBBIAS),   \
		.scan_index = _index,                                 \
		.scan_type = {                                        \
				.sign = 's',                          \
				.realbits = 16,                       \
				.storagebits = 16,                    \
				.shift = 0,                           \
				.endianness = IIO_BE,                 \
			     },                                       \
		.ext_info = inv_ext_info,                             \
	}
```
#### in_accel_x(y_z)index
Positive integer specifying the position of this scan element in the buffer. These values are used in conjuction with *_en* and *_type* values.

## Reading raw values from the sensor
If the device is correctly installed, it will create a file in `/dev/iio:device2` the number corresponds to the location of the device in `/sys/bus/iio/devices/iio:deviceX`.
We can just use `cat /dev/iio:device2` to read the raw data from the IMU. 
{{< notice info >}}
> Before reading the raw data using cat or hexdump, it is necessary to enable the sensor data. The sensor data can be enabled by setting appropriate `in_accel_x,y,z_en` files for accelerometer and `in_anglvel_x,y,z_en` for gyroscope. Furthermore, it is important to enable the buffer by writing 
> ```bash
> echo 1 > scan_elements/enable
> ```
{{< /notice >}}

By using **hexdump** we can read the data from the device.
```bash
00000000  02 c8 03 55 3b b5 00 72  ff c4 ff db 02 d5 04 10  |...U;..r........|
00000010  3b ba 00 7c ff c3 ff da  02 99 03 6a 3b c3 00 79  |;..|.......j;..y|
00000020  ff c2 ff d9 02 e1 03 d0  3b c0 00 79 ff c2 ff d9  |........;..y....|
00000030  02 b7 03 8e 3b d3 00 7b  ff c3 ff da 02 b9 03 fc  |....;..{........|
00000040  3b c6 00 7b ff c2 ff da  02 e5 03 82 3b e6 00 78  |;..{........;..x|
```

{{< notice info >}}
>In the above terminal output, it is important to put `-C` in the command. Normally hexdump is going to respect the endianness of the machine and convert the bytes into little endian format. Inorder to not do that `-C` has to be added to the command.
>The same can be acheived by using `cat /dev/iio:device2 | xxd-`
{{< /notice >}}

The data can be interpreted as follows 
`in_accel_x_index` will show where the x value of Accelerometer is. In the device I have, `in_accel_x_index` shows 0. So the first value is the raw value of the accelerometers x axis. Also the `in_accel_x_index` shows 
```bash
debian@BeagleBone:/sys/bus/iio/devices/iio:device2$ cat scan_elements/in_accel_x_type
be:s16/16>>0
```
We need to shift the raw value 0 bytes and it is considered to be a signed integer. 
```bash
>>> 0x02c8 >> 0
712
```
The above is a raw value and it needs to be converted into $m/sec2$ by multipying it with the [in_accel_scale](#in_accel_scale). 
```bash
>>> 712 * 0.000598 -> the scale by reading the file above.
0.425776
```

