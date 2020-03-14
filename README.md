[![](https://img.shields.io/github/release/jaruba/ha-samsungtv-tizen/all.svg?style=for-the-badge)](https://github.com/jaruba/ha-samsungtv-tizen/releases)
[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg?style=for-the-badge)](https://github.com/custom-components/hacs)
[![](https://img.shields.io/github/license/jaruba/ha-samsungtv-tizen?style=for-the-badge)](LICENSE)
[![](https://img.shields.io/badge/MAINTAINER-%40jaruba-red?style=for-the-badge)](https://github.com/jaruba)
[![](https://img.shields.io/badge/COMMUNITY-FORUM-success?style=for-the-badge)](https://community.home-assistant.io)

# HomeAssistant - SamsungTV Tizen Component

This is a custom component to allow control of SamsungTV devices in [HomeAssistant](https://home-assistant.io). Is a modified version of the built-in [samsungtv](https://www.home-assistant.io/integrations/samsungtv/) with some extra features.<br/>
**This plugin is only for 2016+ TVs model!** (maybe all tizen family)

**Development for this project relies solely on donations, so if you enjoy this component, please consider [becoming a patron](https://www.patreon.com/powder_tech) or [donating](https://powder.media/donate) to ensure it's continued survival.**

# Additional Features:

* Ability to send keys using a native Home Assistant service
* Ability to send chained key commands using a native Home Assistant service
* Supports Assistant commands (Google Home, should work with Alexa too, but untested)
* Extended volume control
* Ability to customize source list at media player dropdown list
* Migrate SamsungCtl to SamsungTvWs 
* Cast video URLs to Samsung TV
* Connect to SmartThings Cloud API for additional features: see TV channel names, see which HDMI source is selected, more key codes to change input source

![N|Solid](https://i.imgur.com/8mCGZoO.png)
![N|Solid](https://i.imgur.com/t3e4bJB.png)


# Installation

### 1. Easy Mode

Install via HACS.


### 2. Manual

Install it as you would do with any homeassistant custom component:

1. Download `custom_components` folder.
2. Copy the `samsungtv_tizen` direcotry within the `custom_components` directory of your homeassistant installation. The `custom_components` directory resides within your homeassistant configuration directory.
**Note**: if the custom_components directory does not exist, you need to create it.
After a correct installation, your configuration directory should look like the following.
    ```
    └── ...
    └── configuration.yaml
    └── custom_components
        └── samsungtv_tizen
            └── __init__.py
            └── media_player.py
            └── websockets.py
            └── shortcuts.py
            └── smartthings.py
            └── upnp.py
            └── exceptions.py
            └── manifest.json
    ```


# Configuration

1. Enable the component by editing the configuration.yaml file (within the config directory as well).
Edit it by adding the following lines:
    ```
    # Example configuration.yaml entry
    media_player:
      - platform: samsungtv_tizen
        host: IP_ADDRESS
        port: 8002
        mac: MAC_ADDRESS
    ```
    **Note**: This is the same as the configuration for the built-in [Samsung Smart TV](https://www.home-assistant.io/integrations/samsungtv/) component.

    ### Custom configuration variables

    **broadcast_address:**<br/>
    (string)(Optional)<br/>
    The ip address of the host to send the magic packet (for wakeonlan) to if the "mac" property is also set.<br/>
    Default value: "255.255.255.255"<br/>
    Example value: "192.168.1.255"<br/>

    **update_method:**<br/>
    (string)(Optional)<br/>
    This change the ping method used for state update. Values: "ping", "websockets" and "smartthings"<br/>
    Default value: "ping"<br/>
    Example value: "websockets"<br/>
    
    **update_custom_ping_url:**<br/>
    (string)(Optional)<br/>
    Use custom endpoint to ping.<br/>
    Default value: PING TO 8001 ENDPOINT<br/>
    Example value: "http://192.168.1.77:9197/dmr"<br/>
    
    **source_list:**<br/>
    (json)(Optional)<br/>
    This contains the KEYS visible sources in the dropdown list in media player UI.<br/>
    Default value: '{"TV": "KEY_TV", "HDMI": "KEY_HDMI"}'<br/>
    You can also chain KEYS, example: '{"TV": "KEY_SOURCES+KEY_ENTER"}'<br/>
    And even add delays (in milliseconds) between sending KEYS, example:<br/>
    '{"TV": "KEY_SOURCES+500+KEY_ENTER"}'<br/>
    Resources: [key codes](./Key_codes.md) / [key patterns](./Key_chaining.md)<br/>
    **Warning: changing input source with voice commands only works if you set the device name in `source_list` as one of the whitelisted words that can be seen on [this page](https://web.archive.org/web/20181218120801/https://developers.google.com/actions/reference/smarthome/traits/modes#mode-settings) (under "Mode Settings")**<br/>
    
    **app_list:**<br/>
    (json)(Optional)<br/>
    This contains the APPS visible sources in the dropdown list in media player UI.<br/>
    Default value: AUTOGENERATED<br/>
    Example value: '{"Netflix": "11101200001", "YouTube": "111299001912", "Spotify": "3201606009684"}'<br/>
    Known lists of App IDs: [List 1](https://github.com/tavicu/homebridge-samsung-tizen/issues/26#issuecomment-447424879), [List 2](https://github.com/Ape/samsungctl/issues/75#issuecomment-404941201)<br/>
    You can also add at the end of every app ID an additional value (separated by "/") that indicate the channel name exposed by SmartThings. This will help identifing the running app if SmartThings API are enabled. This additional value is optional and if not present will be used the app ID to identify the running app.<br/>
    Example value: '{"Netflix": "11101200001/org.tizen.netflix-app", "Prime Video": "3201512006785/org.tizen.ignition", "Spotify": "3201606009684"}'<br/>
    **Note: although this setting is optional, it is highly recommended (for best performance) to set it manually and not rely on the autogenerated list.**<br/>

    **api_key:**<br/>
    (string)(Optional)<br/>
    API Key for the SmartThings Cloud API, this is optional but adds better state handling on, off, channel name, hdmi source, and a few new keys: `ST_TV`, `ST_HDMI1`, `ST_HDMI2`, `ST_HDMI3`, etc.<br/>
    [How to get an API Key for SmartThings](./Smartthings.md)<br/>
    _You must set both an `api_key` and `device_id` to enable the SmartThings Cloud API_<br/>
    
    **device_id:**<br/>
    (string)(Optional)<br/>
    Although this is an optional value, it is mandatory if you've set a SmartThings API Key in order to identify your device in the API.<br/>
    [How to get a device ID from SmartThings](./Smartthings.md)<br/>
    _You must set both an `api_key` and `device_id` to enable the SmartThings Cloud API_<br/>

    **show_channel_number:**<br/>
    (boolean)(Optional)<br/>
    If the SmartThings API is enabled (by settings "api_key" and "device_id"), then the TV Channel Names will show as media titles, by setting this to True the TV Channel Number will also be attached to the end of the media title. (when applicable)

    **scan_app_http:**<br/>
    (boolean)(Optional)<br/>
    Allow you to disable the scan of running app polling port 8001 of the TV to have the refresh of the TV status and running App faster.<br/>
    If the SmartThings API is enabled (by settings "api_key" and "device_id"), then this option is false by default and the scan is diabled. You can enable it setting the value to true.<br/>
    If the SmartThings API is disabled, then this option is always set to true.

2. Reboot Home Assistant
3. Congrats! You're all set!

# Usage

### Known Supported Voice Commands

* Turn on `SAMSUNG-TV-NAME-HERE` (for some older TVs this only works if the TV is connected by LAN cable to the Network)
* Turn off `SAMSUNG-TV-NAME-HERE`
* Volume up on `SAMSUNG-TV-NAME-HERE` (increases volume by 1)
* Volume down on `SAMSUNG-TV-NAME-HERE` (decreases volume by 1)
* Set volume to 50 on `SAMSUNG-TV-NAME-HERE` (sets volume to 50 out of 100)
* Mute `SAMSUNG-TV-NAME-HERE` (sets volume to 0)
* Change input source to `DEVICE-NAME-HERE` on `SAMSUNG-TV-NAME-HERE` (only works if `DEVICE-NAME-HERE` is a whitelisted word from [this page](https://web.archive.org/web/20181218120801/https://developers.google.com/actions/reference/smarthome/traits/modes) under "Mode Settings")

(if you find more supported voice commands, please create an issue so I can add them here)

### Cast to TV

`service: media_player.play_media`

```
{
  "entity_id": "media_player.samsungtv",
  "media_content_type": "url",
  "media_content_id": "FILE_URL",
}
```
_Replace FILE_URL with the url of your file._

### Send Keys
```
service: media_player.play_media
```

```json
{
  "entity_id": "media_player.samsungtv",
  "media_content_type": "send_key",
  "media_content_id": "KEY_CODE",
}
```
**Note**: Change "KEY_CODEKEY" by desired key_code. (also works with key chaining and SmartThings keys: ST_TV, ST_HDMI1, ST_HDMI2, ST_HDMI3, etc.)

Script example:
```
tv_channel_down:
  alias: Channel down
  sequence:
  - service: media_player.play_media
    data:
      entity_id: media_player.samsung_tv55
      media_content_id: KEY_CHDOWN
      media_content_type: "send_key"
```


***Key Codes***
---------------
To see the complete list of known keys, [check this list](./Key_codes.md)


***Key Chaining Patterns***
---------------
Key chaining is also supported, which means a pattern of keys can be set by delimiting the keys with the "+" symbol, delays can also be set in milliseconds between the "+" symbols.

[See the list of known Key Chaining Patterns](./Key_chaining.md)


***Open Browser Page***
---------------

```
service: media_player.play_media
```

```json
{
  "entity_id": "media_player.samsungtv",
  "media_content_type": "browser",
  "media_content_id": "https://www.google.com",
}
```
