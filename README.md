# openhmd-wmr-usb
Trying to get Samsung Odyssey + to work on linux with openhmd


##WIP LOG:

#### History
I can't get my Samsung Odyssey + running in linux with openhmd. The screen stays off but the device sends back IMU data.

I think that a bulk usb transfer cmd must be sent to enable the HDMI display. So I went and captured USB data in Windows and try to look for suspicius things.

### Status

The device sends back a interup when the internal proximity switch sense that you put the headset on your head.
The leftover data is **10014706**. A second interup is sent when the switch is unpressed because you removed your headset. It's **10004606**. It's from endpoint 0x81 

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

```
0000   1b 00 50 60 84 53 0a e6 ff ff 00 00 00 00 09 00   ..P`.S..........
0010   01 04 00 06 00 81 01 04 00 00 00 10 00 46 06      .............F.
```

Also, the onboard camera have to be enabled. Thanks to @pH5 for pointing out the [information](https://github.com/OpenHMD/OpenHMD/issues/200). The way it works, a first cmd is sent to disable the camera stream and a second one is sent to re-enable it.

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

