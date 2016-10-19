# Example mesh application for mbed OS

This application is the simplest one to utilize our mesh networking stack. It just joins your device to the unsecure *6LoWPAN-ND* network.

## Setup

### Download the application

```
mbed import mbed-os-example-mesh-minimal
cd mbed-os-example-mesh-minimal
```

### Change the channel settings (optional)

See the file `mbed_app.json` for an example of defining an IEEE 802.15.4 channel to use.

### Selecting optimal Nanostack configuration

If you want to optimize the flash usage, you need to select a proper configuration for Nanostack. The configuration depends mostly on the preferred use case.

See [6LoWPAN overview](https://docs.mbed.com/docs/arm-ipv66lowpan-stack/en/latest/quick_start_intro/index.html) for the definition of star and mesh networks. These same principles apply also to Thread protocol.

Select the protocol the network is based on:

- 6LoWPAN-ND
- Thread

Select the device role:

- Mesh network. A router. (default)
- Star network. Non routing device. Also known as a host, or sleepy host.

Modify your `mbed_app.json` file to tell which Nanostack build to choose and which configrations to use on [mbed Mesh API](https://github.com/ARMmbed/mbed-os/blob/master/features/nanostack/FEATURE_NANOSTACK/mbed-mesh-api/README.md).

An example of the `mbed_app.json` file:

```
...
        "mesh-type":{
            "help": "options are MESH_LOWPAN, MESH_THREAD",
            "value": "MESH_LOWPAN"
        }
    },
    "target_overrides": {
        "*": {
            "target.features_add": ["NANOSTACK", "LOWPAN_ROUTER", "COMMON_PAL"],
            "mbed-mesh-api.6lowpan-nd-device-type": "NET_6LOWPAN_ROUTER",
            "mbed-mesh-api.thread-device-type": "MESH_DEVICE_TYPE_THREAD_ROUTER",
            "mbed-mesh-api.heap-size": 32000,
            "mbed-trace.enable": false
        }
    }
```

The following tables show the values you should use in the `mbed_app.json` file for your devices in different networks.

- For 6LoWPAN-ND based network use `mesh-type: MESH_LOWPAN`.
- For Thread based network use `mesh-type: MESH_THREAD`.

**mesh-type: MESH_LOWPAN**

|Device role|`target.features_add` value|`mbed-mesh-api.6lowpan-nd-device-type`|
|-----------|-------------------------|------------------------------------|
|Mesh router (default) | LOWPAN_ROUTER | NET_6LOWPAN_ROUTER |
|Non routing device | LOWPAN_HOST | NET_6LOWPAN_HOST |

**mesh-type: MESH_THREAD**

|Device role|`target.features_add` value|`mbed-mesh-api.thread-device-type`|
|-----------|-------------------------|------------------------------------|
|Mesh router (default) | THREAD_ROUTER | MESH_DEVICE_TYPE_THREAD_ROUTER |
|Non routing device | THREAD_END_DEVICE | MESH_DEVICE_TYPE_THREAD_SLEEPY_END_DEVICE |

**Note:** You need to recompile your application after modifying the configurations by issuing the command `mbed compile -c`.

### Requirements for boards without entropy

To run this example application there must be a source of entropy. If the platform you are using does not have
an entropy source you need edit the `mbed_app.json` file to add `NULL_ENTROPY` feature for mbedTLS:

```
"macros": ["MBEDTLS_NO_DEFAULT_ENTROPY_SOURCES",
           "MBEDTLS_TEST_NULL_ENTROPY"],
```

### Compile the application

```
mbed compile -m K64F -t GCC_ARM
```

A binary is generated at the end of the build process.

### Connect the RF shield to the board

By default, we are using the Atmel AT86RF233 and it can be purchased [here](https://firefly-iot.com/product/firefly-arduino-shield-2-4ghz/). Place the shield on top of your board and power it up.

### Program the target

Drag and drop the binary to the target to program the application.

### Update the firmware of the border router

This example supports the following two border routers:

- [k64f-border-router](https://github.com/ARMmbed/k64f-border-router), 6LoWPAN only
- [mbed gateway](https://firefly-iot.com/product/firefly-6lowpan-gateway-2-4ghz/)
 
Read the instructions on updating the firmware of your mbed gateway working as 6LoWPAN [here](https://github.com/ARMmbed/mbed-os-example-client#mbed-gateway).

Both border routers support static and dynamic backhaul configuration. The static configuration is good for testing but the dynamic one works if your network infrastructure is supplying an IPv6 address. Make sure that you use the appropiate mode.

Remember to connect the Ethernet cable between the border router and your home/office router. Then power up the board.

## Testing

As soon as both the border router and the target are up and running you can verify the correct behaviour. Open a serial console and see the IP address obtained by the device.

<span class="notes">**Note:** This application uses the baud rate of 115200.</span>

```
connected. IP = 2001:db8:a0b:12f0::1
```

You can use this IP address to `ping` from your PC and verify that the connection is working correctly.

### Changing the radio driver (optional)

To run a 6LoWPAN-ND network, you need a working RF driver for Nanostack. This example uses the Atmel AT86RF233 by default. 

To change the RF driver:

1. Uninstall the Atmel RF driver.

        mbed remove atmel-rf-driver

2. Install the new driver. (For example, for the FRDM-CR20A radio shield based on the MCR20A device.)

        mbed add mcr20a-rf-driver

3. Recompile your application.

        mbed compile -m K64F -t GCC_ARM

**Note:** Make sure that the `mbed_app.json` file is also updated to reflect the usage of a particular RF driver. For example,

```json
      "radio-type":{
            "help": "options are ATMEL, MCR20",
            "value": "ATMEL"
        },
```
## Important note (Multi-platform support)

mbed OS provides you with total control of the device. However, some defaults are always loaded if you do not provide proper information regarding them. This becomes evident when you switch among platforms. On some platforms, a particular pin might be reserved for a particular functionality (depending upon the MCU) which thus cannot be used generally. If the Ardurino form factor is not available, or the required peripherial is on a different pin, the pins values can be set manually. For example, you can add `"atmel-rf.spi-rst": "D4"` to your `mbed_app.json` file. This will set the SPI_RST pin to D4 of the GPIO.

```json
{
    "target_overrides": {
        "*": {
            "target.features_add": ["NANOSTACK", "LOWPAN_ROUTER", "COMMON_PAL"],
            "atmel-rf.spi-rst": "D4"
        }
    }
}
```

Desired work flow for switching the platform:

1. Check the platform pin map from [mbed Platforms](https://developer.mbed.org/platforms/).
2. Make sure that the desired GPIO pin is free by looking at the data sheet of the particular MCU. Most of the data sheets are available on [mbed Platforms](https://developer.mbed.org/platforms/). 
3. If necessary, change the pin or pins by using the mbed OS config mechanism. To get more information about the configuration system, see the [documentation](https://github.com/ARMmbed/mbed-os/blob/master/docs/config_system.md).
