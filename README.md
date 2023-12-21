# Samsung Air Condition Windfree Home Assistant

This is my configuration in Home Assistant without using the SmartThings integration. My main reason was that I wanted to avoid opening up my home assistant instance to public for the web hook.

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
