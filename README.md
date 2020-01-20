# openhmd-wmr-usb
Trying to get Samsung Odyssey + to work on linux with openhmd


## WIP LOG:

#### History
I can't get my Samsung Odyssey + running in linux with openhmd. The screen stays off but the device sends back IMU data. There seems to be some delay with WMR and when openhmd tries to access device configuration, it will not respond. There has to be more retries and longer wait time for it to work. I also found that sometime starting the exemples programmes a few times in a row will make it work. This has to be fixed as it's a big issue for openhmd-wmr.

I think that a bulk usb transfer cmd must be sent to enable the HDMI display. So I went and captured USB data in Windows and try to look for suspicius things.

### Status

#### USB Identiy

This is from openhmd-wmr-controllers.

```
#define MICROSOFT_VID                   0x045e
#define HOLOLENS_SENSORS_PID            0x0659
#define MOTION_CONTROLLER_PID           0x065b
#define MOTION_CONTROLLER_PID_SAMSUNG   0x065d
```

#### Proximity switch

The device sends back a interup when the internal proximity switch sense that you put the headset on your head.
The leftover data is **10014706**. A second interup is sent when the switch is unpressed because you removed your headset. It's **10004606**. It's from endpoint 0x81, 

<details>
<summary>Proximity switch on</summary>

```
Frame 1: 31 bytes on wire (248 bits), 31 bytes captured (248 bits)
    Encapsulation type: USB packets with USBPcap header (152)
    Arrival Time: Jan 18, 2020 22:19:37.528885000 EST
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1579403977.528885000 seconds
    [Time delta from previous captured frame: 0.000000000 seconds]
    [Time delta from previous displayed frame: 0.000000000 seconds]
    [Time since reference or first frame: 0.000000000 seconds]
    Frame Number: 1
    Frame Length: 31 bytes (248 bits)
    Capture Length: 31 bytes (248 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: usb]
USB URB
    [Source: 4.6.1]
    [Destination: host]
    USBPcap pseudoheader length: 27
    IRP ID: 0xffffe60a53846050
    IRP USBD_STATUS: USBD_STATUS_SUCCESS (0x00000000)
    URB Function: URB_FUNCTION_BULK_OR_INTERRUPT_TRANSFER (0x0009)
    IRP information: 0x01, Direction: PDO -> FDO
    URB bus id: 4
    Device address: 6
    Endpoint: 0x81, Direction: IN
    URB transfer type: URB_INTERRUPT (0x01)
    Packet Data Length: 4
    [bInterfaceClass: Unknown (0xffff)]
Leftover Capture Data: 10014706

```
</details>

```
0000   1b 00 50 60 84 53 0a e6 ff ff 00 00 00 00 09 00   ..P`.S..........
0010   01 04 00 06 00 81 01 04 00 00 00 10 01 47 06      .............G.
```

<details>
<summary>Proximity switch off</summary>

```

Frame 7232: 31 bytes on wire (248 bits), 31 bytes captured (248 bits)
    Encapsulation type: USB packets with USBPcap header (152)
    Arrival Time: Jan 18, 2020 22:19:44.856788000 EST
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1579403984.856788000 seconds
    [Time delta from previous captured frame: 0.001532000 seconds]
    [Time delta from previous displayed frame: 0.001532000 seconds]
    [Time since reference or first frame: 7.327903000 seconds]
    Frame Number: 7232
    Frame Length: 31 bytes (248 bits)
    Capture Length: 31 bytes (248 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: usb]
USB URB
    [Source: 4.6.1]
    [Destination: host]
    USBPcap pseudoheader length: 27
    IRP ID: 0xffffe60a53846050
    IRP USBD_STATUS: USBD_STATUS_SUCCESS (0x00000000)
    URB Function: URB_FUNCTION_BULK_OR_INTERRUPT_TRANSFER (0x0009)
    IRP information: 0x01, Direction: PDO -> FDO
    URB bus id: 4
    Device address: 6
    Endpoint: 0x81, Direction: IN
    URB transfer type: URB_INTERRUPT (0x01)
    Packet Data Length: 4
    [Request in: 2]
    [Time from request: 7.327839000 seconds]
    [bInterfaceClass: Unknown (0xffff)]
Leftover Capture Data: 10004606

```
</details>

```
0000   1b 00 50 60 84 53 0a e6 ff ff 00 00 00 00 09 00   ..P`.S..........
0010   01 04 00 06 00 81 01 04 00 00 00 10 00 46 06      .............F.
```

#### Onboard camera

Also, the onboard camera have to be enabled. Thanks to pH5 for pointing out the [information](https://github.com/OpenHMD/OpenHMD/issues/200). It's a 12-byte bulk data sent to endpoint 0x05. The way it works, a first cmd is sent to disable the camera stream and a second one is sent to re-enable it. Looking at the hex data, it's  : 
```
44 6c 6f 2b 0c 00 00 00 82 00 00 00
```
```
44 6c 6f 2b 0c 00 00 00 81 00 00 00
```

There was a [pull request #208](https://github.com/OpenHMD/OpenHMD/pull/208) by mossvr that added camera support, but was not allowed as it used libusb. It's good as reference and nice on his part. 

<details>
<summary>Disable onboard camera</summary>

```
Frame 15: 39 bytes on wire (312 bits), 39 bytes captured (312 bits)
    Encapsulation type: USB packets with USBPcap header (152)
    Arrival Time: Jan 18, 2020 22:19:38.007902000 EST
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1579403978.007902000 seconds
    [Time delta from previous captured frame: 0.000029000 seconds]
    [Time delta from previous displayed frame: 0.000029000 seconds]
    [Time since reference or first frame: 0.479017000 seconds]
    Frame Number: 15
    Frame Length: 39 bytes (312 bits)
    Capture Length: 39 bytes (312 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: usb]
USB URB
    [Source: host]
    [Destination: 4.2.5]
    USBPcap pseudoheader length: 27
    IRP ID: 0xffffe60a5384c7f0
    IRP USBD_STATUS: USBD_STATUS_SUCCESS (0x00000000)
    URB Function: URB_FUNCTION_BULK_OR_INTERRUPT_TRANSFER (0x0009)
    IRP information: 0x00, Direction: FDO -> PDO
    URB bus id: 4
    Device address: 2
    Endpoint: 0x05, Direction: OUT
    URB transfer type: URB_BULK (0x03)
    Packet Data Length: 12
    [Response in: 16]
    [bInterfaceClass: Unknown (0xffff)]
Leftover Capture Data: 446c6f2b0c00000082000000

```
</details>

```
0000   1b 00 f0 c7 84 53 0a e6 ff ff 00 00 00 00 09 00
0010   00 04 00 02 00 05 03 0c 00 00 00 44 6c 6f 2b 0c
0020   00 00 00 82 00 00 00
```

<details>
<summary>Enable onboard camera</summary>

```
Frame 1423: 39 bytes on wire (312 bits), 39 bytes captured (312 bits)
    Encapsulation type: USB packets with USBPcap header (152)
    Arrival Time: Jan 18, 2020 22:19:38.348793000 EST
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1579403978.348793000 seconds
    [Time delta from previous captured frame: 0.000042000 seconds]
    [Time delta from previous displayed frame: 0.000042000 seconds]
    [Time since reference or first frame: 0.819908000 seconds]
    Frame Number: 1423
    Frame Length: 39 bytes (312 bits)
    Capture Length: 39 bytes (312 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: usb]
USB URB
    [Source: host]
    [Destination: 4.2.5]
    USBPcap pseudoheader length: 27
    IRP ID: 0xffffe60a53bb6860
    IRP USBD_STATUS: USBD_STATUS_SUCCESS (0x00000000)
    URB Function: URB_FUNCTION_BULK_OR_INTERRUPT_TRANSFER (0x0009)
    IRP information: 0x00, Direction: FDO -> PDO
    URB bus id: 4
    Device address: 2
    Endpoint: 0x05, Direction: OUT
    URB transfer type: URB_BULK (0x03)
    Packet Data Length: 12
    [Response in: 1424]
    [bInterfaceClass: Unknown (0xffff)]
Leftover Capture Data: 446c6f2b0c00000081000000

```

</details>

```
0000   1b 00 60 68 bb 53 0a e6 ff ff 00 00 00 00 09 00
0010   00 04 00 02 00 05 03 0c 00 00 00 44 6c 6f 2b 0c
0020   00 00 00 81 00 00 00

```

#### Other communication from host to OpenHMD

Looking at comunication from host to endpoint 0x05, there are not many packets from the time the proximity switch is togled on to when the display is actually lit up. Excluding the 2 packets for the camera, I captured 16 other packets.

They share some similarity as they all start with same 10 bytes. Only the 9th byte of them is different from the camera command. For the rest, they don't have much variations in the 14 first bytes except the 11th byte that flips from 00 to 01. For now, I could assume that they are a sequence of initialization. 

```
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 a4 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 01 00 70 17 ff 00 01 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 e3 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 ff 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 01 00 70 17 ff 00 01 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 ff 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 01 00 70 17 ff 00 01 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 ff 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 f3 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 e1 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 d3 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 ce 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 cc 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 cb 00 00 00
44 6c 6f 2b 12 00 00 00 80 00 00 00 70 17 d0 00 00 00
```

#### Video packets

Stated from pH5 : "While the stream is active, video frames are captured via 616538-byte bulk transfers from endpoint 0x85. These contain 25 packets of 24576 bytes and one 2138-byte packet, with a 32-byte header each. Strip the headers, and the payload contains a 1280x481 8-bit grayscale image of both sensors with some metadata in the first line." 
"Every third frame is brighter, with a long exposure for headset tracking, the others are for controller tracking with a very short exposure. Bytes 6-7 of the first line for each sensor are either 0x2c 0x01 (little endian 300) or 0x00 0x00 depending on the frame type."



