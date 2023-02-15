---
title: "[EN] Power Supply Hanmatek/HM310T"
date: 04/12/2018
draft: false

cover:
    image: cover.png
    alt: "cover"
    relative: true

tags: ["lang-eng"]
---

Good power supply for hobbyists. It is cheap and provides a way to control it from a computer but it has some drawbacks. Let's perform some checks.

## Brief

| Catogory                            | Check | Reason                                                     |
|-------------------------------------|-------|------------------------------------------------------------|
| [Value for money](#value-for-money) | ✅     | ~125€ when this post was written for a good set of feature |
| [USB Support](#usb-support)         | ❌     | Issue on detection + USB IDs badly customized              |
| [Control Protocol](#control-protocol)         | ✅     | Issue on detection + USB IDs badly customized              |

## Value for money



## USB Support

When you plug this power supply into your Linux, the first thing that you can do is the *lsusb* command to see if it has been detected.

```bash
lsusb
# Bus YYY Device XXX: ID 1a86:7523 QinHeng Electronics CH340 serial converter
```

The first check shows that the USB module has NOT been customized, it is still the USB ids of the converter *QinHeng Electronics CH340 serial converter*. The second thing to check is to see if the *serial_short* USB has been modified.

```bash
udevadm info /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
# ...
# N: ttyUSB0
# ...
# E: MAJOR=188
# E: MINOR=0
# E: SUBSYSTEM=tty
# ...
# E: ID_VENDOR_ID=1a86
# E: ID_MODEL_ID=7523
# ...
# E: ID_SERIAL=1a86_USB_Serial
# ...
```

Ugly 🤮! there is no *serial_short*, which means that the ID_SERIAL will be the same if you plug multiple power supplies into your computer and you won't be able to identify it accurately.

### Known Issues: /dev/ttyUSBx is not present in Ubuntu 22.04

On Ubuntu 22, there is a USB conflict, follow this to fix it.

```bash
# EDIT >>> 
/usr/lib/udev/rules.d/85-brltty.rules

# Search for this line and comment it out:
ENV{PRODUCT}=="1a86/7523/*", ENV{BRLTTY_BRAILLE_DRIVER}="bm", GOTO="brltty_usb_run"

# reboot
```

## Control Protocol

This device uses Modbus protocol, here are the register map

**Modbus Registers**

DeviceAddr default = 1 MODBUS slave ID address

| Name           | Address     | Mode | Description                                                                    |
| -------------- | ----------- | ---- | ------------------------------------------------------------------------------ |
| PS_PowerSwitch | 0x01        | R/W  | 0/1  Power output/stop setting                                                 |
| PS_ProtectStat | 0x02        | R    | Bit mask  SCP:OTP:OPP:OCP:OVP                                                  |
| PS_Model       | 0x03        | R    | 3010 (HM310P)                                                                  |
| PS_ClassDetial | 0x04        | R    | value 0x4b58 (19280)                                                           |
| PS_Voltage     | 0x0010 (16) | R    | 2Decimal Voltage display value                                                 |
| PS_Current     | 0x0011      | R    | 3Decimal Current display value                                                 |
| PS_PowerH      | 0x0012      | R    | 3Decimal Power display value 0012H(high 16 bit)/ 0013H( low 16 bit )           |
| PS_PowerL      | 0x0013      | R    | 3Decimal Power display value 0012H(high 16 bit)/ 0013H( low 16 bit )           |
| PS_PowerCal    | 0x0014      | ?    |                                                                                |
| PS_ProtectVol  | 0x0020      | RW   | 2Decimal OVP Set over volate protect value                                     |
| PS_ProtectCur  | 0x0021      | RW   | 2Decimal OCP Set over current protect value                                    |
| PS_ProtectPow  | 0x0022      | RW   | 2Decimal OPP Set over power protect value 0022H(high 16 bit)，023H(low 16 bit) |
| PS_SetVoltage  | 0x0030 (48) | RW   | 2Dec Set voltage                                                               |
| PS_SetCurrent  | 0x0031      | RW   | 3Dec Set current                                                               |
| PS_SetTimeSpan | 0x0032      | RW   | ????                                                                           |
| PS_PowerStat   | 0x8801      | ?    |                                                                                |
| PS_defaultShow | 0x8802      | ?    |                                                                                |
| PS_SCP         | 0x8803      | ?    |                                                                                |
| PS_Buzzer      | 0x8804      | RW   | Buzzer enablement                                                              |
| PS_Device      | 0x9999      | R/W  | Set communication address - SlaveID : 1 default                                |
| PS_SDTime      | 0xCCCC      | ?    |                                                                                |
| PS_UL          | 0xC110      | ?    | 11d / xC111 = 1                                                                |
| PS_UH          | 0xC11E      | ?    | 3200d / xC11F = 1                                                              |
| PS_IL          | 0xC120      | ?    | 21 / xC121=1                                                                   |
| PS_IH          | 0xC12E      | ?    | 10100/ xC12F=1                                                                 |
| PSM_Voltage    | 0x1000      | RW   |                                                                                |
| PSM_Current    | 0x1001      | RW   |                                                                                |
| PSM_TimeSpan   | 0x1002      | R?   |                                                                                |
| PSM_Enable     | 0x1003      | ?    |                                                                                |
| PSM_NextOffset | 0x10        | ?    |                                                                                |

**Sources**

- https://github.com/notkevinjohn/HM310P
- https://github.com/hobbyquaker/hanmatek-hm310p
- https://github.com/mckenm/HanmaTekPSUCmd/wiki/Registers (to get the Modbus registers full list)

