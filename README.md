# hdmi-capture
Notes about capturing HDMI on the Raspberry Pi.

## Purpose:
The purpose of this repository is to collect information on capturing HDMI with a Raspberry Pi. Note that this is not an authoritative guide on HDMI, CSI, or any other topics. This is just a place for me to keep my notes.

## Hardware:
The hardware used for these experiments is a HDMI to MIPI CSI-2 board that was obtained from [AliExpress](https://www.aliexpress.com/item/4000152180240.html). The board contains a Toshiba TC358749XBG HDMI-RX to MIPI CSI-2-TX bridge IC ([datasheet](media/(U18)TC358749XBG_V074.pdf)). The board connects to the Raspberry Pi via the flat flex camera connector.

HDMI to CSI-2 Board |
------------ |
<img src="media/IMG_20200620_142118.jpg" width="500px"> |

Connection to Raspberry Pi |
------------ |
<img src="media/IMG_20200620_142054.jpg" width="500px"> |

**Note:** The Raspberry Pi only exposes 2 lanes of the CSI interface (except for some of the compute modules). As such, this limits the resolution and framerate of the HDMI signal that can be captured.

## Software Configuration:
To enable the capture card, a few changes have to be made to the `/boot` partition of the Raspberry Pi.

1. The amount of memory for the CMA needs to be increased. This is accomplished by adding the following to `/boot/cmdline.txt`
```
cma=128M
```
2. The TC358749XBG overlay needs to be enabled. This is accomplished by appending the following to `/boot/config.txt`
```
dtoverlay=tc358743
```
3. It probably doesn't hurt to increase the amount of memory for the VideoCore. Append the following to `/boot/config.txt`
```
gpu_mem=256
```

The following command can be used to return status information from v4l2:
```
v4l2-ctl --log-status
```

## Sound:
The Toshiba IC does have the ability to receive sound from the HDMI source, and the I2S pins are broken out on the board, but there is evidence to suggest that it does not work properly with the Raspberry Pi. A `tc358743-audio.dtbo` overlay does exist though.

## The EDID:
The EDID is a block of data that the HDMI sink presents to the HDMI source. It lists the formats and timings that are supported by the HDMI sink. Example EDID blocks can be found in [edid/](edid/). Because of the two lane limitation noted above, the maximum resolution and framerate that can be received is 1080P at 50FPS YCbCr 4:2:2, or 1080P at 30FPS RGB24.

The EDID can be set with the following command:
```
v4l2-ctl --set-edid=file=1080P50EDID.txt --fix-edid-checksums
```

## Misc Commands:
When the HDMI link is established, timing information can be checked with:
```
v4l2-ctl --query-dv-timings
```

Timing information can then be set with:
```
v4l2-ctl --set-dv-bt-timings query
```
