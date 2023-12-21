# Samsung Air Conditioner (Windfree) in Home Assistant

This is my configuration in Home Assistant without using the SmartThings integration. My main reason was that I wanted to avoid opening up my home assistant instance to public for the web hook.

Things needed:
- Smartthings Account
- Custom [template_climate](https://github.com/jcwillox/hass-template-climate) extension

### Get a personal access token
Retrieve your personal access token [here](https://account.smartthings.com/tokens) and grant full access to devices.

### configuration.yaml

Some reusable parts to control the AC

```yaml
input_select:
  samsung_ac_optional_mode:
    options:
      - "off"
      - "sleep"
      - "speed"
      - "quiet"
      - "windFree"
      - "windFreeSleep"
    icon: mdi:target

rest_command:
  # device_id, state=[on,off]
  samsung_ac_switch:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "switch",  "command": "{{ state }}"}  ]}'
    content_type: 'application/json; charset=utf-8'
  # device_id, temperature=int
  samsung_ac_set_temperature:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "thermostatCoolingSetpoint",  "command": "setCoolingSetpoint",  "arguments": [ {{ temperature }} ]}  ]}'
    content_type: 'application/json; charset=utf-8'
  # device_id, mode=[cool,heat,wind,dry,auto]
  samsung_ac_mode:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "airConditionerMode",  "command": "setAirConditionerMode",  "arguments": [ "{{ mode }}" ]}  ]}'
    content_type: 'application/json; charset=utf-8'
  # device_id, mode=[off,sleep,speed,quiet,windFree,windFreeSleep]
  samsung_ac_optional_mode:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "custom.airConditionerOptionalMode",  "command": "setAcOptionalMode",  "arguments": [ "{{ mode }}" ]}  ]}'
    content_type: 'application/json; charset=utf-8'
  # device_id, mode=[off,fixed,vertical,horizontal,all]
  samsung_ac_swing_mode:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "fanOscillationMode",  "command": "setFanOscillationMode",  "arguments": [ "{{ mode }}" ]}  ]}'
    content_type: 'application/json; charset=utf-8'
  # device_id, mode=[auto,low,medium,high,turbo]
  samsung_ac_fan_mode:
    url: 'https://api.smartthings.com/v1/devices/{{ device_id }}/commands'
    method: POST
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    payload: '{  "commands": [{  "component": "main",  "capability": "airConditionerFanMode",  "command": "setFanMode",  "arguments": [ "{{ mode }}" ]}  ]}'
    content_type: 'application/json; charset=utf-8'
```

Configuration per Air Conditioning

```yaml
rest:
  - resource_template: 'https://api.smartthings.com/v1/devices/<YOUR_DEVICE_ID>/status'
    method: GET
    headers:
      authorization: !secret smartthings_token
      accept: 'application/json'
      user-agent: 'Home-Assistant'
    scan_interval: 5
    binary_sensor:
      - name: "Living Room AC Status"
        value_template: '{{ value_json.components.main.switch.switch.value }}'
    sensor:
      - name: "Living Room AC Humidity"
        device_class: humidity
        unit_of_measurement: "%"
        value_template: '{{ value_json.components.main.relativeHumidityMeasurement.humidity.value }}'
      - name: "Living Room AC Temperature"
        device_class: temperature
        unit_of_measurement: "°C"
        value_template: '{{ value_json.components.main.temperatureMeasurement.temperature.value }}'
      - name: "Living Room AC Target Temperature"
        device_class: temperature
        unit_of_measurement: "°C"
        value_template: '{{ value_json.components.main.thermostatCoolingSetpoint.coolingSetpoint.value }}'
      - name: "Living Room Optional AC Mode"
        value_template: '{{ value_json.components.main["custom.airConditionerOptionalMode"].acOptionalMode.value }}'
      - name: "Living Room AC Mode"
        value_template: '{{ value_json.components.main.airConditionerMode.airConditionerMode.value }}'
      - name: "Living Room AC Swing Mode"
        value_template: '{{ value_json.components.main.fanOscillationMode.fanOscillationMode.value }}'
      - name: "Living Room AC Fan Mode"
        value_template: '{{ value_json.components.main.airConditionerFanMode.fanMode.value }}'

switch:
  - platform: template
    switches:
      living_room_ac_switch:
        value_template: "{{ is_state('binary_sensor.living_room_ac_status', 'on') }}"
        turn_on:
          service: rest_command.samsung_ac_switch
          data:
            device_id: "<YOUR_DEVICE_ID>"
            state: "on"
        turn_off:
          service: rest_command.samsung_ac_switch
          data:
            device_id: "<YOUR_DEVICE_ID>"
            state: "off"

template:
  select:
    - name: "Living Room AC Optional Mode"
      state: "{{ states('sensor.living_room_optional_ac_mode')}}"
      options: "{{ state_attr('input_select.samsung_ac_optional_mode', 'options') }}"
      select_option:
        - service: rest_command.samsung_ac_optional_mode
          data:
            device_id: "<YOUR_DEVICE_ID>"
            mode: "{{ option }}"

climate:
  - platform: climate_template
    name: Living Room AC
    min_temp: 16
    max_temp: 30
    current_temperature_template: "{{ states('sensor.living_room_ac_temperature') }}"
    current_humidity_template: "{{ states('sensor.living_room_ac_humidity') }}"
    availability_template: "on"
    target_temperature_template: "{{ states('sensor.living_room_ac_target_temperature') }}"
    set_temperature: 
      service: rest_command.samsung_ac_set_temperature
      data:
        device_id: "<YOUR_DEVICE_ID>"
        temperature: "{{ temperature }}"
    modes:
      - "heat"
      - "dry"
      - "cool"
      - "wind"
    hvac_mode_template: "{{ states('sensor.living_room_ac_mode') }}"
    set_hvac_mode:
      service: rest_command.samsung_ac_mode
      data:
        device_id: "<YOUR_DEVICE_ID>"
        mode: "{{ hvac_mode }}"
    fan_modes: ["auto","low","medium","high","turbo"]
    fan_mode_template: "{{ states('sensor.living_room_ac_fan_mode') }}"
    set_fan_mode: 
      service: rest_command.samsung_ac_fan_mode_cool
      data:
        device_id: "<YOUR_DEVICE_ID>"
        mode: "{{ fan_mode }}"
    swing_modes: ["off","fixed","vertical","horizontal","all"]
    swing_mode_template: "{{ states('sensor.living_room_ac_swing_mode') }}"
    set_swing_mode: 
      service: rest_command.samsung_ac_swing_mode_cool
      data:
        device_id: "<YOUR_DEVICE_ID>"
        mode: "{{ swing_mode }}"

```

### UI proposal
<img width="558" alt="ac-ui" src="https://github.com/visiongeist/samsung-ac-windfree-home-assistant/assets/1744831/0151da81-f0ec-49d1-9965-f8a13fc9c58d">

