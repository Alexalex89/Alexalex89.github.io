---

title: Use your heatpump efficiently with sun energy

date: "2024-06-23"

description: Create a Home Assistant automation to load your hot water tank when the sun is shining

---

{{< figure src="photovoltaic-green.png" title="" >}}

## Prerequisites

- Up-to-date [Home Assistant](https://www.home-assistant.io/) installation
- Photovoltaic system that sends production data to Home Assistant
- Heat pump that can be controlled by Home Assistant
- A helper variable (type `bool`) which indicates a photovoltaic electricity surplus
- An integration to get the remaining photovoltaic production of the day ([Forecast Solar integration](https://www.home-assistant.io/integrations/forecast_solar/))
- **Important**: I have a Viessmann heat pump (Vitocal 200a), which might operate differently from yours
- **Note**: You might not be able to use this automation directly (Ctrl+C; Ctrl+V) in your setup, but it should serve as a good starting point.

## How it works

I have been refining my perfect automation for over one and a half years and am now quite satisfied with it. Since our photovoltaic system doesn't have a battery, I was looking for a solution to enable my heat pump to use as much clean energy as possible during sunlight hours. Essentially, I increase the temperature of my hot water tank when certain conditions are met.

When enough energy is available, the automation checks if one of the following is *true* and increases the temperature:
- **A**: The water temperature is more than 4 degrees below the target temperature.
  - This means that new warm water is needed very soon.
- **B**: The temperature is just below the target temperature (1-2 degrees) and the remaining daily energy production is low (< 25 kWh).
  - This suggests there might be time left to produce warm water, but depending on the remaining energy prediction, it is advisable to start right away.
- **C**: The temperature is slightly further below the target temperature (1-4 degrees) and the remaining daily energy production is moderate (< 35 kWh).
  - This suggests there might be little time left to produce warm water, but depending on the remaining energy prediction, it is advisable to start right away.
- **D**: The heat pump is already active and the valve is set to water heating.
  - It's not beneficial for the heat pump to start frequently, so if it's already running, we should increase the temperature now!

My Viessmann heat pump has a functionality called 1xWW (more or less starting manual warm water production, but with a different target temperature). My usual temperature is set to 43 degrees Celsius, and the increased temperature to 50 degrees Celsius. The only condition to start this mode is that the hysteresis of the target temperature is reached, which is set to 5K in my case. In other words, it has to be below (50-5=45) degrees Celsius.

After checking the conditions, the automation waits a few minutes to ensure the energy conditions are still favorable.

## The Script

Here's my script; you might have to adjust the parameters according to your variable names.


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
        entity_id: sensor.heat_pump_mode
        state: Heating and Hot Water
      - condition: state
        entity_id: sensor.heat_pump_mode
        state: Hot Water
  - condition: state
    entity_id: binary_sensor.surplus_power_available
    state: "on"
  - condition: template
    value_template: >-
      {{ (as_timestamp(now()) -
      as_timestamp(states('input_datetime.last_heat_pump_activation'))) > 3600 }}      
  - condition: or
    conditions:
      - condition: template
        value_template: >-
          {{ states('sensor.heat_pump_actual_top')|float <
          (states('sensor.heat_pump_target')|float - 4) }}          
      - condition: and
        conditions:
          - condition: template
            value_template: >-
              {{ states('sensor.heat_pump_actual_top')|float >
              (states('sensor.heat_pump_target')|float - 1) }}              
          - condition: template
            value_template: >-
              {{ states('sensor.heat_pump_actual_top')|float <
              ([states('sensor.heat_pump_target')|float + 2, 45]|min) }}              
          - condition: template
            value_template: >-
              {{ states('sensor.energy_production_today_remaining')|float < 25
              }}              
      - condition: and
        conditions:
          - condition: template
            value_template: >-
              {{ states('sensor.heat_pump_actual_top')|float >
              (states('sensor.heat_pump_target')|float - 4) }}              
          - condition: template
            value_template: >-
              {{ states('sensor.heat_pump_actual_top')|float <
              (states('sensor.heat_pump_target')|float - 1) }}              
          - condition: template
            value_template: >-
              {{ states('sensor.energy_production_today_remaining')|float < 35
              }}              
      - condition: and
        conditions:
          - condition: state
            entity_id: binary_sensor.compressor_status
            state: "on"
          - condition: state
            entity_id: sensor.heat_pump_valve
            state: Hot Water
action:
  - delay:
      minutes: 2
  - if:
      - condition: state
        entity_id: binary_sensor.surplus_power_available
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
      entity_id: input_datetime.last_heat_pump_activation
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
