# hass-dmx
Home Assistant DMX over IP Light Platform

The DMX integration for Home Assistant allows you to send DMX values to an [Art-Net](http://www.art-net.org.uk) capable DMX interface.

The component is a one way integration, and sends UDP packets to the DMX interface. This integration uses no external libraries and requires at least Python version 3.5.

### Prerequisites

You need at least hass >= 0.66 in order to use this component.

### Usage

To use DMX in your installation:
1. Download the [light.py](https://github.com/jnimmo/hass-artnet/raw/master/dmx/light.py) file and save into the *'custom_components/dmx'* directory. (Create a *'custom_components'* folder in the location of your configuration.yaml file, and create a subdirectory *'dmx'* to store this platform)
2. Add the following lines to your `configuration.yaml` file:

```yaml
light:
  - platform: dmx
    host: <IP Address>
    port: 6454
    dmx_channels: 512 
    default_level: 255
    universe: 0
    devices:
      - channel: 1
        name: House lights
        type: dimmer
        transition: 3
      - channel: 2
        name: Hall lights
        type: dimmer
        default_level: 255
      - channel: 3
        name: Stair lights
        type: dimmer
        transition: 3
      - channel: 4
        type: rgb
        name: Entrance LED Strip
        default_rgb: [0,0,150]
      - channel: 7
        type: dimmer
        name: Smoke machine
      - channel: 8
        type: custom_white
        name: Intensity/Temperature Light
        channel_setup: dT
```

Configuration variables:
- **host** (*Required*): Host Art-Net/DMX gateway
- **port** (*Optional*): Defaults to 6454
- **dmx_channels** (*Required*): The number of DMX channels to send a value for (even number between 2 & 512)
- **default_level** (*Required*): Default level for Home Assistant to assume the lights have been set to - in most cases 0 would make sense. Note Home Assistant will not send these values to the gateway until an explicit change is made unless send_levels_on_startup is True.
- **send_levels_on_startup** (*Optional*): Defaults to True if not specified. Setting this to False means Home Assistant will not send any DMX frames until a change is made.
- **universe** (*Optional*): Artnet Universe. Defaults to 0 if not specified.

Device configuration variables:
- **channel** (*Required*): The DMX channel for the light (1-512)
- **name** (*Required*): Friendly name for the light (will also be used for the entity_id)
- **type** (*Required*): 
  - **'dimmer'** (single channel)
  - **'rgb'** (red, green, blue)
  - **'rgbw'** (red, green, blue, white)
  - **'rgbw_auto'** (red, green, blue, automatically calculated white value) 
  - **'drgb'** (dimmer, red, green, blue)
  - **'drgbw'** (dimmer, red, green, blue, white)
  - **'rgbwd'** (red, green, blue, white, dimmer)
  - **'switch'** (single channel 0 or 255)
  - **'custom_white'** (configure dimmer and temperature in any required channel order)
- **default_level** (*Optional*): Default level to assume the light is set to (0-255). 
- **channel_setup** (*For custom_white lights*): String to define channel layout where:
  - d = dimmer (brightness 0 to 255)
  - t = temperature (0 = warm, 255 = cold)
  - T = temperature (255 = warm, 0 = cold)
  - h = warm white value (scaled for brightness)
  - c = cool white value (scaled for brightness)
  
Please use [light_profiles.csv](https://www.home-assistant.io/components/light/#default-turn-on-values) if you want to specify a default colour or brightness to be used when turning the light on in HA.
- **default_rgb** (*Optional*): Default colour to give to Home Assistant for the light in the format [R,G,B]
- **white_level** (*Optional*): Default white level for RGBW lights (0-255)
- **transition** (*Optional*): Set a default fade time for transitions. Transition times specified through the turn_on / turn_off service calls in Home Assistant will override this behaviour. 

Supported features:
- Transition time can be specified through services to fade to a colour (for RGB fixtures) or value. This currently is set to run at 40 frames per second. Multiple fades at the same time seem to be possible.
- Brightness: Once a channel is turned on brightness can be controlled through the Home Assistant interface.
- White level: For RGB lights with a separate white LED this controls the white LED. This can be automatically controlled using the colour wheel on 'rgbw_auto' lights, or manually with 'rgbw'
- Color temperature: For dual channel warm white/cool white fixtures this tunes the white temperature.

Limitations:
- DMX frames must send values for all channels in a universe. If you have other channels which are controlled by a different device or lighting desk, set Home Assistant to default to 0 values; and set your Art-Net device to merge on highest value rather than most recent update. This means channels could be controlled from either the desk or Home Assistant.

### Support for other hardware

- Simple, FTDI-chip based USB2DMX cables can be made working with this component through a [UDP proxy implemented in C](https://gist.github.com/zonque/10b7b7183519bf7d3112881cb31b6133).
- DMX King eDMX1
- Enttec ODE MK2

**Art-Net™ Designed by and Copyright Artistic Licence Holdings Ltd**
