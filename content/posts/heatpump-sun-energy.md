---

title: Use your heatpump efficiently with sun energy

date: "2024-06-23"

description: Create a Home Assistant automation to load your hot water tank when the sun is shining

---

## Prerequisites

- up-to-date [Home Assistant](https://www.home-assistant.io/) installation
- photovoltaic system which sends production data to Home Assistant
- heat pump which can be controlled by Home Assistant
- a helper variable (type `bool`) which indicates photovoltaic electricity surplus
- an integration to get remaining photovoltaic production of the day ([Forecast Solar integration](https://www.home-assistant.io/integrations/forecast_solar/))
- **important**: I have a Viessmann heat pump (vitocal 200a), which might work different than yours
- **I'm afraid you won't be able to use this Automation as Ctrl+C; Ctrl+V with your setup, but at least you will have a good starting point.**

## How it works

I was working on (my) perfect automation for over one and a half years and now I'm quite happy with it for a while.
We don't have a battery for our photovoltaic system, so I was trying to find a solution which enables my heat pump to use as much clean energy as possible during sun-hours.
Basically it's just that I increase the temperature of my hot water tank when some conditions are right.

When enough energy is available, the automation checks if one of the following is *true* and increases the temperature:
- **A**: The water temperature is more than 4 degrees below the target temperature.
	- this means that new warm water is needed very soon.
- **B**: The temperature is just below the target temperature (1-2 degrees) and the remaining daily energy production is low (< 25 kWh).
	- this means that there might be time left to produce warm water, but depending on the remaining energy prediction it is a good idea to start right away.
- **C**: The temperature is slightly further below the target temperature (1-4 degrees) and the remaining daily energy production is moderate (< 35 kWh).
	- this means that there might be little time left to produce warm water, but depending on the remaining energy prediction it is a good idea to start right away.
- **D**: The heat pump is already active and the valve is set to water heating.
	- it's not good for the heat pump to be started frequently. So if it's already running, we should increase the temperature now!

My Viessmann heat pump has a functionality called 1xWW (more or less starting a manual warm water production, but with a different target temperature). My usual temperature is set to 43 degrees Celsius and the increased temperature to 50 degrees Celsius. The only condition to start this mode is that the hysteresis of the target temperature is reached, which is set to 5K in my case. In other words, it has to be below (50-5=45) degrees Celsius.

After checking the conditions, the automation is waiting for a few minutes to check if the energy conditions are still good.

## The script

Here's my script, you might have to adjust the params according to your variable names.

```yaml
alias: Heatpump WW
description: ""
trigger:
  - platform: time_pattern
    minutes: /2
condition:
  - condition: or
    conditions:
      - condition: state
        entity_id: sensor.wp_betriebsart
        state: Heizen und Warmwasser
      - condition: state
        entity_id: sensor.wp_betriebsart
        state: Warmwasser
  - condition: state
    entity_id: binary_sensor.strom_uebrig_ohne_wp_neu
    state: "on"
  - condition: template
    value_template: >-
      {{ (as_timestamp(now()) -
      as_timestamp(states('input_datetime.last_wp_ww_activation'))) > 3600 }}
  - condition: or
    conditions:
      - condition: template
        value_template: >-
          {{ states('sensor.wp_ww_ist_oben')|float <
          (states('sensor.wp_ww_soll')|float - 4) }}
      - condition: and
        conditions:
          - condition: template
            value_template: >-
              {{ states('sensor.wp_ww_ist_oben')|float >
              (states('sensor.wp_ww_soll')|float - 1) }}
          - condition: template
            value_template: >-
              {{ states('sensor.wp_ww_ist_oben')|float <
              ([states('sensor.wp_ww_soll')|float + 2, 45]|min) }}
          - condition: template
            value_template: >-
              {{ states('sensor.energy_production_today_remaining')|float < 25
              }}
      - condition: and
        conditions:
          - condition: template
            value_template: >-
              {{ states('sensor.wp_ww_ist_oben')|float >
              (states('sensor.wp_ww_soll')|float - 4) }}
          - condition: template
            value_template: >-
              {{ states('sensor.wp_ww_ist_oben')|float <
              (states('sensor.wp_ww_soll')|float - 1) }}
          - condition: template
            value_template: >-
              {{ states('sensor.energy_production_today_remaining')|float < 35
              }}
      - condition: and
        conditions:
          - condition: state
            entity_id: binary_sensor.wp_status_verdichter
            state: "on"
          - condition: state
            entity_id: sensor.wp_ventil
            state: WW
action:
  - delay:
      minutes: 2
  - if:
      - condition: state
        entity_id: binary_sensor.strom_uebrig_ohne_wp_neu
        state: "off"
    then:
      - stop: ""
  - service: mqtt.publish
    enabled: true
    data:
      qos: 0
      retain: false
      topic: openv/set1xWWein
      payload: "1"
  - service: input_datetime.set_datetime
    data:
      entity_id: input_datetime.last_wp_ww_activation
      datetime: "{{ now().isoformat() }}"
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
mode: single
trace:
  stored_traces: 720

```
