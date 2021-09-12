----
----
# _WORK IN PROGRESS_
----
----

# RAK4631-Kit-2-RAK1910-RAK1904-RAK1906
| <img src="./assets/RAK-Whirls.png" alt="Modules" width="150"> | <img src="./assets/rakstar.jpg" alt="RAKstar" width="100"> |    
| :-: | :-: |     
This is an example code for WisBlock GNSS tracker with RAK1910 GNSS module, RAK1904 ACC sensor and RAK1906 environment sensor. 

## _REMARK 1_
The WisBlock Kit 2 comes without the RAK1906. But if you want full RAK5205/7205 compatibility you need to order it separate.

- [RAK1906](https://store.rakwireless.com/collections/wisblock-sensor/products/rak1906-bme680-environment-sensor)

## _REMARK 2_
It is based on my low power event driven example [RAK4631-Quick-Start-Examples](https://github.com/beegee-tokyo/RAK4631-Quick-Start-Examples)

----

# Hardware used
- [RAK4631](https://docs.rakwireless.com/Product-Categories/WisBlock/RAK4631/Overview/) WisBlock Core module
- [RAK5005-O](https://docs.rakwireless.com/Product-Categories/WisBlock/RAK5005-O/Overview/) WisBlock Base board
- [RAK1910](https://docs.rakwireless.com/Product-Categories/WisBlock/RAK1910/Overview/) WisBlock Sensor GNSS module
- [RAK1904](https://docs.rakwireless.com/Product-Categories/WisBlock/RAK1904/Overview/) WisBlock Sensor acceleration sensor
- [RAK1906](https://docs.rakwireless.com/Product-Categories/WisBlock/RAK1906/Overview/) WisBlock Sensor environment module (optional)

## Power consumption
The application does switch off the GPS module and the MCU and LoRa transceiver go into sleep mode between measurement cycles to save power. I could measure a sleep current of 40uA of the whole system. 

----

# Software used
- [PlatformIO](https://platformio.org/install)
- [Adafruit nRF52 BSP](https://docs.platformio.org/en/latest/boards/nordicnrf52/adafruit_feather_nrf52832.html)
- [Patch to use RAK4631 with PlatformIO](https://github.com/RAKWireless/WisBlock/blob/master/PlatformIO/RAK4630/README.md)
- [SX126x-Arduino LoRaWAN library](https://github.com/beegee-tokyo/SX126x-Arduino)
- [TinyGPSPlus](https://platformio.org/lib/show/1655/TinyGPSPlus)
- [SparkFun LIS3DH Arduino Library](https://platformio.org/lib/show/1401/SparkFun%20LIS3DH%20Arduino%20Library)
- [Adafruit BME680 Library](https://platformio.org/lib/show/1922/Adafruit%20BME680%20Library) (optional)

## _REMARK_
The libraries are all listed in the **`platformio.ini`** and are automatically installed when the project is compiled.

----

# Setting up LoRaWAN credentials
The LoRaWAN settings can be defined in three different ways. 
- Over BLE with [My nRF52 Toolbox](https://play.google.com/store/apps/details?id=tk.giesecke.my_nrf52_tb)
- Over USB with [AT Commands](./AT-Commands.md)
- Hardcoded in the sources (_**ABSOLUTELY NOT RECOMMENDED**_)

## 1) Setup over BLE
Using the [My nRF52 Toolbox](https://play.google.com/store/apps/details?id=tk.giesecke.my_nrf52_tb) you can connect to the WisBlock over BLE and setup all LoRaWAN parameters like
- Region
- OTAA/ABP
- Confirmed/Unconfirmed message
- ...

More details can be found in the [My nRF52 Toolbox repo](https://github.com/beegee-tokyo/My-nRF52-Toolbox/blob/master/README.md)

The device is advertising over BLE only the first 30 seconds after power up and then again for 15 seconds after wakeup for measurements. The device is advertising as **`RAK-GNSS-xx`** where xx is the BLE MAC address of the device.

## 2) Setup over USB port
Using the AT command interface the WisBlock can be setup over the USB port.

A detailed manual for the AT commands are in [AT-Commands.md](./AT-Commands.md)

## _REMARK_
The AT command format used here is _**NOT**_ compatible with the RAK5205/RAK7205 AT commands.

## 3) Hardcoded LoRaWAN settings
`void api_set_credentials(void);`
This informs the API that hard coded LoRaWAN credentials will be used. If credentials are sent over USB or from My nRF Toolbox, the received credentials will be ignored. _**It is strongly suggest NOT TO USE hard coded credentials to avoid duplicate node definitions**_    
If hard coded LoRaWAN credentials are used, they must be set before this function is called. Example:    
```c++
g_lorawan_settings.auto_join = false;							// Flag if node joins automatically after reboot
g_lorawan_settings.otaa_enabled = true;							// Flag for OTAA or ABP
memcpy(g_lorawan_settings.node_device_eui, node_device_eui, 8); // OTAA Device EUI MSB
memcpy(g_lorawan_settings.node_app_eui, node_app_eui, 8);		// OTAA Application EUI MSB
memcpy(g_lorawan_settings.node_app_key, node_app_key, 16);		// OTAA Application Key MSB
memcpy(g_lorawan_settings.node_nws_key, node_nws_key, 16);		// ABP Network Session Key MSB
memcpy(g_lorawan_settings.node_apps_key, node_apps_key, 16);	// ABP Application Session key MSB
g_lorawan_settings.node_dev_addr = 0x26021FB4;					// ABP Device Address MSB
g_lorawan_settings.send_repeat_time = 120000;					// Send repeat time in milliseconds: 2 * 60 * 1000 => 2 minutes
g_lorawan_settings.adr_enabled = false;							// Flag for ADR on or off
g_lorawan_settings.public_network = true;						// Flag for public or private network
g_lorawan_settings.duty_cycle_enabled = false;					// Flag to enable duty cycle (validity depends on Region)
g_lorawan_settings.join_trials = 5;								// Number of join retries
g_lorawan_settings.tx_power = 0;								// TX power 0 .. 15 (validity depends on Region)
g_lorawan_settings.data_rate = 3;								// Data rate 0 .. 15 (validity depends on Region)
g_lorawan_settings.lora_class = 0;								// LoRaWAN class 0: A, 2: C, 1: B is not supported
g_lorawan_settings.subband_channels = 1;						// Subband channel selection 1 .. 9
g_lorawan_settings.app_port = 2;								// Data port to send data
g_lorawan_settings.confirmed_msg_enabled = LMH_UNCONFIRMED_MSG; // Flag to enable confirmed messages
g_lorawan_settings.resetRequest = true;							// Command from BLE to reset device
g_lorawan_settings.lora_region = LORAMAC_REGION_AS923_3;		// LoRa region
// Inform API about hard coded LoRaWAN settings
api_set_credentials();
```

_**REMARK!**_    
Hard coded credentials must be set in `void setup_app(void)`!

----

# Packet data format
The packet data is made compatible with the [RAK5205/RAK7205](https://docs.rakwireless.com/Product-Categories/WisTrio/RAK7205-5205/Overview). A detailed explanation and encoders for TTN and Chirpstack can be found in the [RAK5205 Documentation](https://docs.rakwireless.com/Product-Categories/WisTrio/RAK7205-5205/Quickstart/#decoding-sensor-data-on-chirpstack-and-ttn)

----

# Compiled output
The compiled files are located in the [./Generated](./Generated) folder. Each successful compiled version is named as      
**`WisBlock_GNSS_Vx.y.z_YYYYMMddhhmmss`**    
x.y.z is the version number. The version number is setup in the [./platformio.ini](./platformio.ini) file.    
YYYYMMddhhmmss is the timestamp of the compilation.

The generated **`.hex`** file can be used as well to update the device over BLE using either [My nRF52 Toolbox repo](https://github.com/beegee-tokyo/My-nRF52-Toolbox/blob/master/README.md) or [Nordic nRF Toolbox](https://play.google.com/store/apps/details?id=no.nordicsemi.android.nrftoolbox) or [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp)

----

# Debug options 
Debug output can be controlled by defines in the **`platformio.ini`**    
_**LIB_DEBUG**_ controls debug output of the SX126x-Arduino LoRaWAN library
 - 0 -> No debug outpuy
 - 1 -> Library debug output (not recommended, can have influence on timing)    

_**MY_DEBUG**_ controls debug output of the application itself
 - 0 -> No debug outpuy
 - 1 -> Application debug output

_**CFG_DEBUG**_ controls the debug output of the nRF52 BSP. It is recommended to keep it off

## Example for no debug output and maximum power savings:

```ini
[env:wiscore_rak4631]
platform = nordicnrf52
board = wiscore_rak4631
framework = arduino
build_flags = 
	; -DCFG_DEBUG=2
	-DSW_VERSION_1=1 ; major version increase on API change / not backwards compatible
	-DSW_VERSION_2=0 ; minor version increase on API change / backward compatible
	-DSW_VERSION_3=0 ; patch version increase on bugfix, no affect on API
	-DLIB_DEBUG=0    ; 0 Disable LoRaWAN debug output
	-DMY_DEBUG=0     ; 0 Disable application debug output
	-DNO_BLE_LED=1   ; 1 Disable blue LED as BLE notificator
lib_deps = 
	beegee-tokyo/SX126x-Arduino
	mikalhart/TinyGPSPlus
	adafruit/Adafruit BME680 Library
	sparkfun/SparkFun LIS3DH Arduino Library
	https://github.com/beegee-tokyo/WisBlock-API
extra_scripts = pre:rename.py
```