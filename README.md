# SamsungTV-RS232-EX-LINK
Random stuff learned about EX-LINK via RS232

The EX-Link port plugs into a 3.5mm keystone wall jack, to 15m of Solid copper 4 core Station Wire, to a MAX232, to ESP32 PoE ESPHome module, mounted in a 42u rack.  

![IMG_4221](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/7c473eed-0ca8-4167-87b7-3281c3cc9de1)
![IMG_4223](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/3e8b505a-59b8-4f0d-aaf0-c6e451c54d24)
![IMG_E4176](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/16469793-a31a-49a4-b129-2014be62d19a)
![IMG_E4238](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/4cb91043-254f-4c83-9ed6-74dd7b29a0e5)
![IMG_E4232](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/9ca035bb-92ea-410a-bf94-e73481519ea2)
![IMG_E4222](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/ff5feadf-7d4c-4637-856c-0486cb7b7aaf)
![IMG_E4245](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/4cd5539d-ecaf-4caa-9e0d-0a9c0c99229b)

If you need to find out a code from your remote enable debug on the EX-Link port

![Screenshot 2024-04-26 0528282debug](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/44c3dfce-815d-4f73-afde-49bef692be0e)

Connect your RS232 cable to the EX-Link port then open Putty or other terminal Use 115200 baud rate and port.

![Img](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/5eaff231-71ae-4da2-b211-29c634a73439)

**Note:** _Putty can't translate hex so you will see nothing in the console when not running debug. Use terraterm or something else that supports serial hex I/O._

### Trigger Service Menu

Power Off -> Mute -> 1,8,2 -> Power On

### Service Menu Options

Control -> Sub Option

RS-232 Jack -> Left = Debug

Serial Log On/Off = On

Watchdog = Off

Restart the TV (Turn it off and on again)

### Serial Debug

When you press the button on the remote you will see these keyVal
```
[A_APPCM/Fatal] 1626 : @@@[KEY_PROCESS] [DTVInputService] t_OnEvent type [20] , keyVal [0x79] <= The value we want
[A_APPCM/Fatal] 1627 : @@@[KEY_PROCESS] Send Background App [rootapp] Key [121] bSync [TRUE]
[A_APPCM/Fatal] 1628 : @@@[KEY_PROCESS] Current App[rootapp] bConsumed [0]
[A_APPCM/Fatal] 1629 : @@@[KEY_PROCESS] Send Background App [mbr] Key [121] bSync [TRUE]
```
That keyVal is the button pushed.

Now you just have to compute the checksum

0x08 + 0x22 + 0x0D + 0x00 + 0x00 + 0x79 = 0xB0

0x100 - 0xB0 = 0x50

The final command:
0x08,0x22,0x0D,0x00,0x00,0x79,0x50

If we send this to the EX-LINK port we should get:
0C 03 F1

Which means the command was sent OK. You should also see the SmartHub come up on this UN58H5203 generation of 2016 TV.

Don't forget to go back in to the Service menu and reverse the changes and restart the TV or RS232 Commands won't work on the EX-LINK port.

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/2e6a7f84-8c82-4723-b18b-5601c65a2c38)

The Excel spreadsheets above will help you decypher most of the commands. Check at the bottom of the tizen one as there are more than 1 TV year sheets in that one not just Tizen.

You can grab 3.3v compatible Dual Port RS232 to TTL converter boards from Aliexpress that should do the same as the board I soldered together above which would make it easier and cheaper. Search terms: RS232 SP3232 TTL to RS232 Module RS232 to TTL MAX3232. The dual port one looks like this:

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/868a4305-6c6c-4636-8be6-193df3529b43) 

You can also get Serial to 3.5mm jack adapters on there and modules with a DB9 connector built in just make sure you get the right pinout as they can swap the tip and ring pins on some.

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/d4bfea2f-e834-4b1a-b0c4-9cc572199946)
![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/281e7620-c94b-4a4b-bfc3-e410f62914e4)

I personally like the route I chose as then I can extend the RS232 cable side for long runs. If you use the above method you will sort of be limited by the connections to be right at the TV.

My goal was to replicate this IR Remote fully in ESPHome Via the EX-LINK port instead of using IR.

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/2dab7b65-653f-4376-9c29-6078426cff6e)

ESPHome YAML
```yaml
esphome:
  name: esp32-s3-n16r8-001
  friendly_name: ESP32-S3-N16R8-001
  platformio_options:
    board_build.flash_mode: dio

esp32:
  board: esp32-s3-devkitc-1
  #variant: esp32s3
  flash_size: 16MB
  framework:
    #type: arduino
    type: esp-idf
    version: recommended
    sdkconfig_options:
      # Need to set a s3 compatible board for the adf-sdk to compile
      # board specific code is not used though
      CONFIG_ESP32_S3_BOX_BOARD: "y"

# Enable logging
logger:
  #hardware_uart: uart0
  #level: VERBOSE

psram:
  mode: octal
  speed: 80MHz

# Enable Home Assistant API
api:
  encryption:
    key: !secret encryption_key010

ota:
  password: !secret ota_pass010

wifi:
  ssid: !secret wifi_ssid2
  password: !secret wifi_password
  use_address: !secret use_address_wifi010
  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "ESP32-S3-N16R8-01FallbackHotspot"
    password: !secret fallbackhotspot010

captive_portal:

web_server:

uart:
  baud_rate: 9600
  tx_pin: 17
  rx_pin: 7
  id: uart_bus
  debug:
    direction: BOTH
    dummy_receiver: true
    sequence:
      - lambda: UARTDebug::log_hex(direction, bytes, ':');
      - lambda: if (direction == uart::UART_DIRECTION_RX) id(get_status).publish_state(format_hex_pretty(bytes));
      - lambda: >
          if (direction == uart::UART_DIRECTION_RX) {
            // The get status hex should be 16 bits long
            if (bytes.size() == 16) {
              // Power State
              // 03.0C.F1.03.0C.F5.08.F0.00.00.00.F1.05.00.00.0E (16)
              if (bytes[8] == 0x00) {
                if (bytes[12] == 0x05) {
                  id(power_status).publish_state("On");
                } else if (bytes[12] == 0x04) {
                  id(power_status).publish_state("Stand-By");
                } else if (bytes[12] == 0x08) {
                  id(power_status).publish_state("Off");
                }
              // Volume Status
              // 03.0C.F1.03.0C.F5.08.F0.01.00.00.F1.07.00.00.0B (16)
              } else if (bytes[8] == 0x01) {
                id(volume_status).publish_state(bytes[12]);
              // Mute Status
              } else if (bytes[8] == 0x02) {
                if (bytes[12] == 0x00) {
                  id(mute_status).publish_state("Off");
                } else if (bytes[12] == 0x01) {
                  id(mute_status).publish_state("On");
              }
              // Channel Status
              //                                     Channel/Major/Minor (x12=18,x01=1,x02=2)
              // 03.0C.F1.03.0C.F5.08.F0.03.00.00.F1.12.01.02.FB (16)
              } else if (bytes[8] == 0x03) {
                uint8_t channelprimary = bytes[12];
                uint8_t channelmajor = bytes[13];
                uint8_t channelminor = bytes[14];
                id(channel_status).publish_state(to_string(channelprimary)+"-"+to_string(channelmajor)+"-"+to_string(channelminor));
              // Source Status
              } else if (bytes[8] == 0x04) {
                // 03.0C.F1.03.0C.F5.08.F0.04.00.00.F1.00.00.00.0F (16)
                if (bytes[12] == 0x00) {
                  id(source_status).publish_state("TV");
                // 03.0C.F1.03.0C.F5.08.F0.04.00.00.F1.39.00.00.D6 (16)
                } else if (bytes[12] == 0x39) {
                  id(source_status).publish_state("HDMI1");
                // 03.0C.F1.03.0C.F5.08.F0.04.00.00.F1.3A.00.00.D5 (16)
                } else if (bytes[12] == 0x3A) {
                  id(source_status).publish_state("HDMI2");
                // 03.0C.F1.03.0C.F5.08.F0.04.00.00.F1.1C.00.00.F3 (16)
                } else if (bytes[12] == 0x1C) {
                  id(source_status).publish_state("AV1");
                // 03.0C.F1.03.0C.F5.08.F0.04.00.00.F1.29.00.00.E6 (16)
                } else if (bytes[12] == 0x29) {
                  id(source_status).publish_state("Component1");
                }
              }
            }
          }

text_sensor:
  - platform: template
    id: "get_status"
    name: "Get Status"
  - platform: template
    id: "power_status"
    name: "Power Status"
  - platform: template
    id: "source_status"
    name: "Source Status"
  - platform: template
    id: "channel_status"
    name: "Channel Status"

sensor:
  - platform: template
    id: "volume_status"
    name: "Volume Status"
    accuracy_decimals: 0

binary_sensor:
  - platform: template
    id: "mute_status"
    name: "Mute Status"

button:
  - platform: restart
    name: "Restart"
    id: but_rest
  - platform: uart
    uart_id: uart_bus
    name: "Power Toggle"
    data: [0x08, 0x22, 0x00, 0x00, 0x00, 0x00, 0xD6]
  - platform: uart
    uart_id: uart_bus
    name: "Power On" # 08 22 00 00 00 02 D4
    data: [0x08, 0x22, 0x00, 0x00, 0x00, 0x02, 0xD4]
  - platform: uart
    uart_id: uart_bus
    name: "Power Off"
    data: [0x08, 0x22, 0x00, 0x00, 0x00, 0x01, 0xD5]
  - platform: uart
    uart_id: uart_bus
    name: "Volume Up"
    data: [0x08, 0x22, 0x01, 0x00, 0x01, 0x00, 0xD4]
  - platform: uart
    uart_id: uart_bus
    name: "Volume Down"
    data: [0x08, 0x22, 0x01, 0x00, 0x02, 0x00, 0xD3]
  - platform: uart
    uart_id: uart_bus
    name: "Mute"
    data: [0x08, 0x22, 0x02, 0x00, 0x00, 0x00, 0xD4]
  - platform: uart
    uart_id: uart_bus
    name: "TV"
    data: [0x08, 0x22, 0x0A, 0x00, 0x00, 0x00, 0xCC]
  - platform: uart
    uart_id: uart_bus
    name: "HDMI 1"
    data: [0x08, 0x22, 0x0A, 0x00, 0x05, 0x00, 0xC7]
  - platform: uart
    uart_id: uart_bus
    name: "HDMI 2"
    data: [0x08, 0x22, 0x0A, 0x00, 0x05, 0x01, 0xC6]
  - platform: uart
    uart_id: uart_bus
    name: "AV1"
    data: [0x08, 0x22, 0x0A, 0x00, 0x01, 0x00, 0xCB]
  - platform: uart
    uart_id: uart_bus
    name: "Component1"
    data: [0x08, 0x22, 0x0A, 0x00, 0x03, 0x00, 0xC9]
  - platform: uart
    uart_id: uart_bus
    name: "Source - Smart Hub"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x79, 0x50]
  - platform: uart
    uart_id: uart_bus
    name: "Source - Netflix"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0xD7, 0xF2]
  - platform: uart
    uart_id: uart_bus
    name: "Source - Prime"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0xD6, 0xF3]
  - platform: uart
    uart_id: uart_bus
    name: "Source - YouTube"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0xD8, 0xF1]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 1"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x04, 0xC5]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 2"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x05, 0xC4]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 3"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x06, 0xC3]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 4"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x08, 0xC1]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 5"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x09, 0xC0]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 6"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x0A, 0xBF]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 7"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x0C, 0xBD]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 8"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x0D, 0xBC]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 9"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x0E, 0xBB]
  - platform: uart
    uart_id: uart_bus
    name: "Digit 0"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x11, 0xB8]
  - platform: uart
    uart_id: uart_bus
    name: "Digit -"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x23, 0xA6]
  - platform: uart
    uart_id: uart_bus
    name: "Channel Up"
    data: [0x08, 0x22, 0x03, 0x00, 0x01, 0x00, 0xD2]
  - platform: uart
    uart_id: uart_bus
    name: "Channel Down"
    data: [0x08, 0x22, 0x03, 0x00, 0x02, 0x00, 0xD1]
  - platform: uart
    uart_id: uart_bus
    # 10-1
    name: "Channel1"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x0A, 0xC8]
  - platform: uart
    uart_id: uart_bus
    # 14-3
    name: "Channel2"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x0E, 0xC4]
  - platform: uart
    uart_id: uart_bus
    # 18-1
    name: "Channel3"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x12, 0xC0]
  - platform: uart
    uart_id: uart_bus
    # 19-1
    name: "Channel4"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x13, 0xBF]
  - platform: uart
    uart_id: uart_bus
    # 20-1
    name: "Channel5"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x14, 0xBE]
  - platform: uart
    uart_id: uart_bus
    # 31-1
    name: "Channel6"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x01F, 0xB3]
  - platform: uart
    uart_id: uart_bus
    # 69-1
    name: "Channel7"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x45, 0x8D]
  - platform: uart
    uart_id: uart_bus
    name: "Input Source Toggle"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x01, 0xC8]
  - platform: uart
    uart_id: uart_bus
    name: "Previous Channel"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x13, 0xB6]
  - platform: uart
    uart_id: uart_bus
    name: "Channel List"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x6B, 0x5E]
  - platform: uart
    uart_id: uart_bus
    name: "Guide"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x4F, 0x7A]
  - platform: uart
    uart_id: uart_bus
    name: "Tools"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x1F, 0xAA]
  - platform: uart
    uart_id: uart_bus
    name: "Info Display"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x1F, 0xAA]
  - platform: uart
    uart_id: uart_bus
    name: "Menu"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x1A, 0xAF]
  - platform: uart
    uart_id: uart_bus
    name: "Up"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x60, 0x69]
  - platform: uart
    uart_id: uart_bus
    name: "Down"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x61, 0x68]
  - platform: uart
    uart_id: uart_bus
    name: "Right"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x62, 0x67]
  - platform: uart
    uart_id: uart_bus
    name: "Left"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x65, 0x64]
  - platform: uart
    uart_id: uart_bus
    name: "Enter OK"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x68, 0x61]
  - platform: uart
    uart_id: uart_bus
    name: "Return Back"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x58, 0x17]
  - platform: uart
    uart_id: uart_bus
    name: "Exit"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x2D, 0x9C]
  - platform: uart
    uart_id: uart_bus
    name: "Red"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x6C, 0x5D]
  - platform: uart
    uart_id: uart_bus
    name: "Green"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x14, 0xB5]
  - platform: uart
    uart_id: uart_bus
    name: "Yellow"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x15, 0xB4]
  - platform: uart
    uart_id: uart_bus
    name: "Blue"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x16, 0xB3]
  - platform: uart
    uart_id: uart_bus
    name: "Picture Size"
    data: [0x08, 0x22, 0x0B, 0x0A, 0x01, 0x05, 0xBB]
  - platform: uart
    uart_id: uart_bus
    name: "E-Manual"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x3F, 0x8A]
  - platform: uart
    uart_id: uart_bus
    name: "Search"
    #data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x93, 0x36]
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x73, 0x56]
  - platform: uart
    uart_id: uart_bus
    name: "Keypad"
    data: [0x08, 0x22, 0x0B, 0x0A, 0x01, 0x05, 0xBB]
  - platform: uart
    uart_id: uart_bus
    name: "MTS"
    #data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x2B, 0x9E]
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x00, 0xC9]
  - platform: uart
    uart_id: uart_bus
    name: "Caption"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x25, 0xA4]
  - platform: uart
    uart_id: uart_bus
    name: "STB"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0xE0, 0xE9]
  - platform: uart
    uart_id: uart_bus
    name: "Play"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x47, 0x82]
  - platform: uart
    uart_id: uart_bus
    name: "Pause"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x4A, 0x7F]
  - platform: uart
    uart_id: uart_bus
    name: "Stop"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x46, 0x83]
  - platform: uart
    uart_id: uart_bus
    name: "Rewind"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x45, 0x84]
  - platform: uart
    uart_id: uart_bus
    name: "Fast Forward"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x48, 0x81]
  - platform: uart
    uart_id: uart_bus
    name: "Skip Back"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x50, 0x79]
  - platform: uart
    uart_id: uart_bus
    name: "Skip Forward"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x4E, 0x7B]
  - platform: uart
    uart_id: uart_bus
    name: "Record"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x49, 0x80]
# Get Command:
# TV Reaction
# Success / Fail _F1_ _Success_
# Data Length    _8_
# Command        _F0_ _0_ _0_ _0_
# Data Value     _5_ _0_ _0_
# Category       _Power_
# Current State  _Normal Mode_
  - platform: uart
    uart_id: uart_bus
    name: "Power Get"
    data: [0x08, 0x22, 0xF0, 0x00, 0x00, 0x00, 0xE6]
  - platform: uart
    uart_id: uart_bus
    name: "Volume Get"
    data: [0x08, 0x22, 0xF0, 0x01, 0x00, 0x00, 0xE5]
  - platform: uart
    uart_id: uart_bus
    name: "Mute Get"
    data: [0x08, 0x22, 0xF0, 0x02, 0x00, 0x00, 0xE4]
  - platform: uart
    uart_id: uart_bus
    name: "Channel Get"
    data: [0x08, 0x22, 0xF0, 0x03, 0x00, 0x00, 0xE3]
  - platform: uart
    uart_id: uart_bus
    name: "Source Get"
    data: [0x08, 0x22, 0xF0, 0x04, 0x00, 0x00, 0xE2]
```

Home Assistant YAML

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/31f45263-b61c-4165-a8af-ae2d35dc902c)

Requires HACS -> https://my.home-assistant.io/redirect/hacs_repository/?repository=android-tv-card&owner=Nerwyn&category=Plugin

https://github.com/Nerwyn/android-tv-card

```yaml
type: custom:android-tv-card
remote_id: remote.tvsamsung58
title: Samsung TV 58
custom_actions:
  '0':
    icon: mdi:numeric-0
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_0
  '1':
    icon: mdi:numeric-1
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_1
  '2':
    icon: mdi:numeric-2
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_2
  '3':
    icon: mdi:numeric-3
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_3
  '4':
    icon: mdi:numeric-4
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_4
  '5':
    icon: mdi:numeric-5
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_5
  '6':
    icon: mdi:numeric-6
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_6
  '7':
    icon: mdi:numeric-7
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_7
  '8':
    icon: mdi:numeric-8
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_8
  '9':
    icon: mdi:numeric-9
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_digit_9
  dash:
    icon: mdi:minus
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_dash
  up:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_up
  down:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_down
  left:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_left
  right:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_right
  center:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_enter_ok
  power:
    icon: mdi:power
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_power_toggle
  source:
    icon: mdi:star-four-points-circle-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_input_source_toggle
  hub:
    icon: mdi:apps
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_source_smart_hub
  previous:
    icon: mdi:page-previous-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_previous_channel
  list:
    icon: mdi:view-list
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel_list
  menu:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_menu
  back:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_return_back
  tools:
    icon: mdi:tools
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_tools
  info:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_info_display
  guide:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_guide
  exit:
    icon: mdi:exit-run
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_exit
  volume_mute:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_mute
  volume_up:
    icon: mdi:arrow-up-bold
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_volume_up
  volume_down:
    icon: mdi:arrow-down-bold
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_volume_down
  ch_up:
    icon: mdi:arrow-up-bold
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel_up
  ch_down:
    icon: mdi:arrow-down-bold
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel_down
  netflix:
    icon: mdi:netflix
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_source_netflix
  youtube:
    icon: mdi:youtube
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_source_youtube
  primevideo:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_source_prime
  tv:
    icon: mdi:antenna
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_tv
  hdmi1:
    icon: mdi:nintendo-switch
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_hdmi1
  hdmi2:
    icon: mdi:hdmi-port
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_hdmi2
  av1:
    icon: mdi:video-input-component
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_av1
  component1:
    icon: mdi:video-input-component
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_component1
  channel1:
    icon: mdi:robot-love
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel1
  channel2:
    icon: mdi:numeric-1-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel2
  channel3:
    icon: mdi:numeric-2-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel3
  channel4:
    icon: mdi:numeric-3-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel4
  channel5:
    icon: mdi:numeric-4-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel5
  channel6:
    icon: mdi:numeric-5-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel6
  channel7:
    icon: mdi:numeric-6-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_channel7
  prog_red:
    icon: mdi:alpha-a-box-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_red
  prog_green:
    icon: mdi:alpha-b-box-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_green
  prog_yellow:
    icon: mdi:alpha-c-box-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_yellow
  prog_blue:
    icon: mdi:alpha-d-box-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_blue
  manual:
    icon: mdi:book-open-blank-variant-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_e_manual
  search1:
    icon: mdi:search-web
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_search
  keypad:
    icon: mdi:open-in-app
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_keypad
  picturesize:
    icon: mdi:resize
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_picture_size
  mts:
    icon: mdi:globe-model
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_mts
  cc:
    icon: mdi:closed-caption-outline
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_caption
  rewind:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_rewind
  pause:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_pause
  fast_forward:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_fast_forward
  record:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_record
  play:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_play
  stop:
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_stop
rows:
  - - power
    - source
  - - volume_up
    - volume_mute
    - ch_up
  - - volume_down
    - list
    - ch_down
  - - menu
    - hub
    - guide
  - - tools
    - up
    - info
  - - left
    - enter
    - right
  - - back
    - down
    - exit
  - - prog_red
    - prog_green
    - prog_yellow
    - prog_blue
  - - manual
    - search1
    - keypad
  - - picturesize
    - mts
    - cc
  - - tv
    - hdmi1
    - hdmi2
    - av1
    - component1
  - - channel1
    - channel2
    - channel3
    - channel4
    - channel5
    - channel6
    - channel7
  - - rewind
    - pause
    - fast_forward
  - - record
    - play
    - stop
  - - 1
    - 2
    - 3
  - - 4
    - 5
    - 6
  - - 7
    - 8
    - 9
  - - dash
    - 0
    - previous
```

## Get Commands

```
# Get Command:
# TV Reaction
# Success / Fail _F1_ _Success_
# Data Length    _8_
# Command        _F0_ _0_ _0_ _0_
# Data Value     _5_ _0_ _0_
# Category       _Power_
# Current State  _Normal Mode_
```

Get Power On

`8 22 F0 0 0 0 E6`

Power Return:

`3 C F1 3 C F5 8 F0 0 0 0 F1 5 0 0`

`3 C F1` Command received (FF = fail)

`3 C F5` Getting Data

`8` Data Length

`F0 0 0 0` Repeats back command you sent

`F1` Success

`5` TV Power State is Normal Mode (On)

`0` Dummy data

`0` Dummy data

Status:

`F1` Success

`FF` Fail

Data Output:

`4` Stand-By

`5` Normal Mode

`8` Off

Get Volume Command

`8 22 F0 1 0 0 E5`

Get Volume Return

`3 C F1 3 C F5 8 F0 1 0 0 F1 C 0 0`

Hex C => Volume is at 12

![image](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/f24a4371-0b15-4710-aa06-d7f8e4de77d0)

```yaml
type: entities
title: Living Room TV
entities:
  - entity: button.esp32_s3_n16r8_001_power_get
    name: Power Get
  - entity: sensor.esp32_s3_n16r8_001_power_status
    name: Power Status
  - entity: button.esp32_s3_n16r8_001_source_get
    name: Source Get
  - entity: sensor.esp32_s3_n16r8_001_source_status
    name: Source Status
  - entity: button.esp32_s3_n16r8_001_channel_get
    name: Channel Get
  - entity: sensor.esp32_s3_n16r8_001_channel_status
    name: Channel Status
  - entity: button.esp32_s3_n16r8_001_volume_get
    name: Volume Get
  - entity: sensor.esp32_s3_n16r8_001_volume_status_2
    name: Volume Status
  - entity: button.esp32_s3_n16r8_001_mute_get
    name: Mute Get
  - entity: binary_sensor.esp32_s3_n16r8_001_mute_status
    name: Mute Status
```

A full write-up is in the 19 tizen spreadsheet above. Click the GetStatus tab in the sheet to find out more.

Some TVs may not support this but seems to work fine on a 2016 that I was using. Otherwise check your connections TX RX swap etc.

If you don't have a 3.5mm EX-Link port on your SamsungTV there still may be hope with the USB port. One may be able to convert to an RS232 port changing some settings in the service menu. It uses the D+ and D- of the USB port for UART TX and RX.

![USB](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/257a9de1-25f7-49be-a6f4-625b8cbcbfbb)
![UART_USB](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/a32bc4ca-2736-4045-8821-e8473f93e650)


With newer modules you can also just use the network to control but this method seems to be way more stable on this TV.
