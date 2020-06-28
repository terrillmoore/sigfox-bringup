# Step-by-step log of Sigfox bringup

## Required materials

- STM32 LoRa Discovery Kit B-L072Z-LRWAN1

- SMA antenna for above for 915 or 868 MHz

- Sigfox gateway

- Sigfox RF test kit (SDR adapter)

## Required Installations

### Source code

Make a directory (I chose `c:/mcci/projects/shetland/s`) and clone the following repositories into the directory.

- git@github.com:disk91/itsdk-example-murata-lora.git
- git@github.com:disk91/itsdk-example-sigfox-sx1276.git
- git@github.com:disk91/stm32-it-sdk.git

### Ac6 build system

- login and register at https://www.openstm32.org

- Download the IDE -- at time of writing, this was [here](https://www.openstm32.org/Downloading+the+System+Workbench+for+STM32+installer).

- Install it. I installed in my compressed `c:/tools` directory as `c:/tools/ac6`. It will show up in your Win10 menus as <image src="assets/ac6-menu-item-win10.png" alt="System Workbench for STM32" width="200px" height="42px" align="middle">.

- Open it up (it's Eclipse, be patient).

- It will prompt with this:

   <image src="assets/Eclipse-start.png" alt="Eclipse startup screen" width="500px">

   I accepted the default; you have to remember this, as this is where your projects get placed.

- Follow the instructions in the video https://www.youtube.com/watch?v=VEWrr4t8ujo -- it's got some errors, and you'll find that the Murata LoRa project hasn't been updated.  Key moments:

  - https://youtu.be/VEWrr4t8ujo?t=107 shows how to import the projects. File>Import>Existing Projects into Workspace, and point to your example directory from above.

  - You need to link the project to the SDK, which you do https://youtu.be/VEWrr4t8ujo?t=137 by expanding the project in the project explorer. Right click on the project top-level; then `New>Folder`, `Advanced>>`, `[x] Link to alternate location`, and select your `stm32-it-sdk` directory.

  **Very important:** his example repositories already have links to an SDK, but they're wrong. So you have to delete them and re-add them. There are two places this must be done.

  **Very important:** you must work with the release configuration, but the default configuration is debug.

  - Right click the project, then `Properties>C/C++ General/Paths and Symbols`. Select the `Source Location` tab on the right panel.
    - select Release configuration.
    - delete the old stm32sk src link; it will be the third or fourth line.
    - click `Add Folder...`
    - expand `stm32-it-sdk` and add `src`.
    - **Don't click apply yet**.

  - Select `Properties>C/C++ Build>Settings`
    - Select `Tool Settings` tab
    - Select `MCU GCC Compiler>Includes`.
    - Again, remove the old `stm32 sdk/inc` entry, as it's for his system not yours (select it, and then press the icon with the red 'x' right above).
    - Now ad the new, using the icon with the green '+' right above, and select the SDK `inc` directory.

- Then apply and say OK.

- Change the configuration to `Release` and build.

- The rest of the information can be ignored.

## Modify hardware

Refer to https://github.com/aureleq/muRataSigfox.

- Move JP9
- short SB13
- short SB26

![Image of dev board changes](https://github.com/aureleq/muRataSigfox/blob/master/img/kithw.jpg)

## Build the project

You'll get several warnings. You can select the Console tab to watch progress. At the end, I got:

```console
Building target: murata-sigfox.elf
Invoking: MCU GCC Linker
arm-none-eabi-gcc -mcpu=cortex-m0plus -mthumb -mfloat-abi=soft -L"C:\Users\tmm\workspace\murata-sigfox\Drivers\LIBS" -specs=nosys.specs -specs=nano.specs -T"../STM32L072CZYx_FLASH.ld" -Wl,-Map=output.map -Wl,--gc-sections -o "murata-sigfox.elf" @"objects.list"  -lSgfxCoreV231_CM0_GCC -lSgfxSTModemSx1276V123_CM0_GCC -lSgfxAddonV020_CM0_GCC -lSgfxCmacV100_CM0_GCC -lm
Finished building target: murata-sigfox.elf

make --no-print-directory post-build
Generating hex and Printing size information:
arm-none-eabi-objcopy -O binary "murata-sigfox.elf" "murata-sigfox.bin"
arm-none-eabi-size "murata-sigfox.elf"
   text	   data	    bss	    dec	    hex	filename
  54088	    264	   6960	  61312	   ef80	murata-sigfox.elf


13:26:56 Build Finished (took 30s.507ms)
```

And for my warnings:

```console
Description	Resource	Path	Location	Type
#warning "sigfox_lowlevel file contains non portable code" [-Wcpp]	sigfox_lowlevel.c	/murata-sigfox/stm32-it-sdk/Src/drivers/sx1276	line 37	C/C++ Problem
#warning "We could optimize long stop with going to sleep..." [-Wcpp]	sigfox_lib_api.c	/murata-sigfox/stm32-it-sdk/Src/drivers/sx1276	line 117	C/C++ Problem
#warning "We may define the pin association with ADC properly for this MCU" [-Wcpp]	adc.c	/murata-sigfox/stm32-it-sdk/Src/stm32l_sdk/adc	line 326	C/C++ Problem
#warning NOPULL from ST_CODE on SIgfox impl, lets see if this could consume energy ?!? was PULLDWN [-Wcpp]	murata_cmwx1zzabz.c	/murata-sigfox/stm32-it-sdk/Src/drivers/sx1276	line 196	C/C++ Problem
#warning the LoRaStack was GPIO_INTERRUPT_RISING_PULLDWN [-Wcpp]	murata_cmwx1zzabz.c	/murata-sigfox/stm32-it-sdk/Src/drivers/sx1276	line 127	C/C++ Problem
implicit declaration of function 'itsdk_print_state'; did you mean 'itsdk_restart'? [-Wimplicit-function-declaration]	console.c	/murata-sigfox/stm32-it-sdk/Src/it_sdk/console	line 171	C/C++ Problem
implicit declaration of function 'log_error'; did you mean 'strerror'? [-Wimplicit-function-declaration]	sdk_config.c	/murata-sigfox/stm32-it-sdk/Src/it_sdk/eeprom	line 92	C/C++ Problem
unused variable 'ret' [-Wunused-variable]	project_main.c	/murata-sigfox/Core/Src	line 76	C/C++ Problem
```

## Downloading

I  was able to install the STLINK  drivers after I did all the normal setup (drivers, etc. from ST). Then I said Project: Run As..>AC6 application. Followng that, blinky stopped and I got different light patterns.. and it seems to work.

## Connecting UARTS

There are two UARTs.

### UART1

UART1 is used for management. You must connect to UART1 with a 3.3V 3-wire UART. Use a Raspberry Pi-compatible unit.

| STM32 Pin | Function | Connector | Pi Cable | Notes
|:---------:|:--------:|:---------:|:--------:|--------
| PB6       | TX       | CN3-11    | Yellow   | The [ST manual][1] says it's on PA9, but this is up to software, and the released software puts it on PB6.
| PA10      | RX       | CN3-21    | Orange   |
| GND       | GND      | CN3-12    | Black    |

Set the terminal emulator to run at 9600 baud, N, 8, 1.  Enable local echo.

After boot, enter '`?`' for help.

Then enter the password "`changeme`" and press enter. Press '?' again, and observe that there are many more commands available.

### UART2

| STM32 Pin | Function | Connector | Notes
|:---------:|:--------:|:---------:|-------
| PA2       | TX       | CN3-22    | Also available via STLINK, so no need to wire up unless STLINK isn't in use.
| PA3       | RX       | CN3-23    | Also available via STLINK, so no need to wire up unless STLINK isn't in use.

UART2 is bridged via the STLINK to a CDC ACM port.

## Testing

With the default software, everything is configured for EU868. So the first thing to do is to change the Sigfox region.

## Configuring for US

Enter configuration mode as described [above](#uart1).

| Step | Command     | Comment               | Reference
|-----:|-------------|-----------------------|------------
|  1.  | `SC:A:02`   | Set device to RC2     | https://support.sigfox.com/docs/radio-configuration
|  2.  | `SC:R:0100` | Set region to US915.  | Source code, `stm32-it-sdk/inc/it_sdk/config_defines.h`, line 342
|  3.  | `S`         | Save configuration    | Online help
|  4.  | `R`         | Reboot device         | Online help

Verify on USART2 log that device uses 902 MHz for the uplink.

## Ordering credentials

### Creating a Product

- login to [build.sigfox.com](https://build.sigfox.com/).
- Create a product (if not done already)
- Say "it's not based on sigfox ready"
- Call it "murata-stm32-demo", and associate with MCCI Corporation
- Click on the "edit" button on the right.
- Fill in the following fields:
  - Model name (already set).
  - Description: B-L072Z-LRWAN1 from ST Micro, with demo software from https://github.com/disk91/itsdk-example-sigfox-sx1276
  - Commercial name: N/A
  - Sector: For all sectors
  - Hardware version: D-03
  - Radio configurations types: Single radio configuration
  - Radio configurations: RC2 - USA, Canada, Mexico, Brazil,
  - Antenna type: 1/4 wave
  - Communication mode: uplink and downlink
  - Payload encryption activated: yes.
  - Power supply: battery
  - Autonomy: 1-3 months
  - Accessibility: Replaceable by user
  - Form factor: AAA
  - Battery capacity: 1000 mAh
  - Number of batteries: 3
  - Nominal voltage: 4.5
  - Minimum voltage 3.3
  - Maximum voltage 5
  - I have read and taken into account the Security Guidelines [yes]
  - I declare that no time synchronization mechanisms have been implemented on the product
  - I declare that pseudo-random frequency series for UL transmissions is unique for each end-point.
  - Select "temperature sensor"
- Save

### Ordering Credentials

- Once product is created, you can click on "Sigfox Network Credentials"
- Order 5 credentials, and say "For Engineering Use".

This seems to take a while -- I tried on a Saturday, and things were still pending several hours later. However, we have a couple Sigfox SDR adapters, so I was able to proceed ([see below](#testing-with-the-sdr-adapter)

### Decrypting Credentials

See https://build.sigfox.com/development#network-credentials

- Download the decryption tool: https://api.build.sigfox.com/files/5c4b2e498344e36dca5e92e8
- Build it using WSL Ubuntu.
- Create a git repo for tracking.

   ```bash
   git init . && git symbolic-ref HEAD refs/head/main
   printf "%s\n" .vscode AESd >> .gitignore
   git add .
   git commit -m "Inital import"
   ```

## Testing with the SDR adapter

The Sigfox SDR ("software defined radio") adapter ([here][2]) allows a PC to function as a special-purpose Sigfox gateway, and allows you to test without dealing with network credentials.

There are two software packages supporting the SDR adapter.

- The Sigfox Network Emulator (SNE)
- The Radio Signal Analyzer (RSA)

SNE is simply an application that runs on Windows or Linux. RSA is a full boot image, and requires that you boot a VM into Ubuntu. Although RSA is interesting, we'll be using SNE.

It was tricky to get the SDR adapter to work with VMware. On my Surface Pro 6, I had to plug into the root port; it wouldn't work at all with VMware when plugged into a hub. Then I had to launch the app several times in the VM. The device went through a number of personalities during that process. I had to repeatedly tell VMware "please attach to the VM"; possibly the device ID was changing. And during that time the app would time out and I'd have to restart.

The SDR adapter started out as a DFU device, but ended up as the following (from USB View):

```console
Device Descriptor:
bcdUSB:             0x0200
bDeviceClass:         0x00
bDeviceSubClass:      0x00
bDeviceProtocol:      0x00
bMaxPacketSize0:      0x40 (64)
idVendor:           0x2CC1
idProduct:          0x0001
bcdDevice:          0x0100
iManufacturer:        0x01
0x0409: "SIGF0X"
iProduct:             0x02
0x0409: "NED... {CONTAINS UNICODE L'\0'!}"
iSerialNumber:        0x03
0x0409: "0000007138BA444900594F57F00ADB3C"
bNumConfigurations:   0x01


Configuration Descriptor:
wTotalLength:       0x0039
bNumInterfaces:       0x03
bConfigurationValue:  0x01
iConfiguration:       0x04
0x0409: "Streaming"
bmAttributes:         0x80 (Bus Powered )
MaxPower:             0x32 (100 Ma)

Interface Descriptor:
bInterfaceNumber:     0x00
bAlternateSetting:    0x00
bNumEndpoints:        0x01
bInterfaceClass:      0xFF
bInterfaceSubClass:   0xFF
bInterfaceProtocol:   0xFF
iInterface:           0x05
0x0409: "RX Stream"

Endpoint Descriptor:
bEndpointAddress:     0x81
Transfer Type:        Bulk
wMaxPacketSize:     0x0200 (512)
bInterval:            0x00

Interface Descriptor:
bInterfaceNumber:     0x01
bAlternateSetting:    0x00
bNumEndpoints:        0x01
bInterfaceClass:      0xFF
bInterfaceSubClass:   0xFF
bInterfaceProtocol:   0xFF
iInterface:           0x06
0x0409: "TX Data... {CONTAINS UNICODE L'\0'!}"

Endpoint Descriptor:
bEndpointAddress:     0x01
Transfer Type:        Bulk
wMaxPacketSize:     0x0200 (512)
bInterval:            0x00

Interface Descriptor:
bInterfaceNumber:     0x02
bAlternateSetting:    0x00
bNumEndpoints:        0x01
bInterfaceClass:      0xFF
bInterfaceSubClass:   0xFF
bInterfaceProtocol:   0xFF
iInterface:           0x06
0x0409: "TX Data... {CONTAINS UNICODE L'\0'!}"

Endpoint Descriptor:
bEndpointAddress:     0x02
Transfer Type:        Bulk
wMaxPacketSize:     0x0200 (512)
bInterval:            0x00

ConnectionStatus: DeviceConnected
Current Config Value: 0x00
Device Bus Speed:     High
Device Address:       0x30
Open Pipes:              0
```

### Connecting the SDR adapter to the B-L072Z-LRWAN1

The SDR kit has a 40 dB attenuator in the box. (It my case, it rolled under the cardboard, so it helped that I watched the [RSA demo video][3] and knew to look for it -- even though the RSA video really isn't about the SNE, and doesn't mention SNE at all, the hardware setup is clear.)  The SDR adapter kit also comes with a short SMA to SMA cable.

- Remove the rubber boots from the attenuator
- Attach the attenuator to the SDR adapter
- Use the SMA-to-SMA cable to attach to the STM32 board (instead of the antenna)

![An image of the setup](assets/SNE-setup-1.jpg)

### Making SNE Work

Once SNE comes up, it pops up a browser window connected to a local web server that is running the app. You also get the usual Windows firewall messages.

- Configure SNE for RC2:

  ![Configuration](assets/SNE-config.png)

- Configure the device for the desired region (e.g., [for US](#configuring-for-us)).

- Enter the device ID (by default, `00D206AF`) into the SNE

  ![Device Entry](assets/SNE-device-config.png)

- At the SNE, Disable authentication (not sure if this is needed, but it seemed to help)

- At the device UART1 port, log in (if it's timed out), then enter the additional commands:

  | Step | Command     | Comment                     | Reference
  |-----:|-------------|-----------------------------|------------
  |  1.  | `SC:9:600`  | Set bit rate to 600         | Match UI on SNE
  |  2.  | `SC:B:1`    | Select "public" operation.  | Help from FAQ "why isn't SNE working?"
  |  3.  | `S`         | Save configuration          | Online help
  |  4.  | `R`         | Reboot device               | Online help

- Check the messages tab, should see something like this:

  ![SNE successful uplink](assets/SNE-success.png)

## References

[Hardware resources](https://www.st.com/en/evaluation-tools/b-l072z-lrwan1.html#resource):

- [Schematics](https://www.st.com/resource/en/schematic_pack/b-l072z-lrwan1_sch.zip) -- we have a rev D-03 board, and the schematics on line are for rev D-01, close enough.
- [Board reference manual][1]

[1]: https://www.st.com/resource/en/user_manual/dm00329995-discovery-kit-for-lorawan-sigfox-and-lpwan-protocols-with-stm32l0-stmicroelectronics.pdf

[2]: https://build.sigfox.com/sdr-dongle

[3]: https://youtu.be/BnFHdZRMUjc