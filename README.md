# SamsungTV-RS232-EX-LINK
Random stuff learned about EX-LINK via RS232
![IMG_4221](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/7c473eed-0ca8-4167-87b7-3281c3cc9de1)
![IMG_4223](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/9b0ea4d1-829f-4284-946b-985256ec034a)
![IMG_E4238](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/4cb91043-254f-4c83-9ed6-74dd7b29a0e5)
![IMG_E4232](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/9ca035bb-92ea-410a-bf94-e73481519ea2)
![IMG_E4222](https://github.com/NonaSuomy/SamsungTV-RS232-EX-LINK/assets/1906575/ff5feadf-7d4c-4637-856c-0486cb7b7aaf)
If you need to find out a code from your remote enable debug on the EX-Link port

Connect your RS232 cable to the EX-Link port then open Putty or other terminal Use 115200 baud rate and port.

Power Off -> Mute -> 1,8,2 -> Power On
Control -> Sub Option
RS-232 Jack -> Left = Debug
Serial Log On/Off = On
Watchdog = Off
Restart the TV (Turn it off and on again)


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

The Excel spreadsheets above will help you decypher most of the commands. Check at the bottom of the tizen one as there are more than 1 year sheets in that one not just Tizen.

My goal was to replicate this IR Remote fully in ESPHome
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
    after:
      delimiter: "\n"
    sequence:
      - lambda: UARTDebug::log_string(direction, bytes);
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
    name: "Power On"
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
    name: "HDMI 1 Switch"
    data: [0x08, 0x22, 0x0A, 0x00, 0x05, 0x00, 0xC7]
  - platform: uart
    uart_id: uart_bus
    name: "HDMI 2 PC"
    data: [0x08, 0x22, 0x0A, 0x00, 0x05, 0x01, 0xC6]
  - platform: uart
    uart_id: uart_bus
    name: "AV1"
    data: [0x08, 0x22, 0x0A, 0x00, 0x01, 0x00, 0xCB]
  - platform: uart
    uart_id: uart_bus
    name: "COMPONENT"
    data: [0x08, 0x22, 0x0A, 0x00, 0x03, 0x00, 0xC9]
  - platform: uart
    uart_id: uart_bus
    name: "Source - Smart Hub"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x79, 0x50]
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
    name: "OTA 10-1"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x0A, 0xC8] #The second last digit is basically the channel number converted to HEX
  - platform: uart
    uart_id: uart_bus
    name: "OTA 69-1"
    data: [0x08, 0x22, 0x04, 0x00, 0x00, 0x45, 0x8D] #The second last digit is basically the channel number converted to HEX
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
    name: "Info (Display)"
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
    name: "Enter (OK)"
    data: [0x08, 0x22, 0x0D, 0x00, 0x00, 0x68, 0x61]
  - platform: uart
    uart_id: uart_bus
    name: "Return (Back)"
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
    name: "Picture Size: Screen Fit"
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
# Success / Fail ___ ___________
# Data Length    ___
# Command        ___ ___ ___ ___
# Data Value     ___ ___ ___
# Category       _______________
# Current State  _______________
  - platform: uart
    uart_id: uart_bus
    name: "Get Power"
    data: [0x08, 0x22, 0xF0, 0x00, 0x00, 0x00, 0xE6]
  - platform: uart
    uart_id: uart_bus
    name: "Get Volume"
    data: [0x08, 0x22, 0xF0, 0x01, 0x00, 0x00, 0xE5]
  - platform: uart
    uart_id: uart_bus
    name: "Get Mute"
    data: [0x08, 0x22, 0xF0, 0x02, 0x00, 0x00, 0xE4]
  - platform: uart
    uart_id: uart_bus
    name: "Get Channel"
    data: [0x08, 0x22, 0xF0, 0x03, 0x00, 0x00, 0xE3]
  - platform: uart
    uart_id: uart_bus
    name: "Get Source"
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
        entity_id: button.esp32_s3_n16r8_001_digit
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
  DAZN:
    icon: dazn
    tap_action:
      action: call-service
      service: media_player.select_source
      data:
        entity_id: media_player.samsung_tv
        source: DAZN
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
  ota:
    icon: mdi:antenna
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_ota
  hdmi1:
    icon: mdi:nintendo-switch
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_hdmi_1_switch
  hdmi2:
    icon: mdi:hdmi-port
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_hdmi_2_pc
  ota0:
    icon: mdi:robot-love
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_ota_10_1
  ota1:
    icon: mdi:numeric-1-box
  ota2:
    icon: mdi:numeric-2-box
  ota3:
    icon: mdi:numeric-3-box
  ota4:
    icon: mdi:numeric-4-box
  ota5:
    icon: mdi:numeric-5-box
  ota6:
    icon: mdi:numeric-6-box
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.esp32_s3_n16r8_001_ota_69_1
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
        entity_id: button.esp32_s3_n16r8_001_picture_size_screen_fit
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
  - - ota
    - hdmi1
    - hdmi2
  - - ota0
    - ota1
    - ota2
    - ota3
    - ota4
    - ota5
    - ota6
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
