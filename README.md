# easee_charger_HA_Node-RED
Contains a Node-RED flow for easee charger lite. I use not only dynamic charging, but also manual charging and timed charging through HA dashboard.



-------------========== ADD THE FOLLOWING IN HA ==========-------------

Helpers needed in HA (if you change the names, please remember to change in Node-RED also):
input_boolean.opladning_afbrudt
input_boolean.afbrudt_overskuds_opladning
input_boolean.prioriter_elbil_opladning
input_boolean.oplad_efter_solnedgang
input_number.energi_buffer_i_watt (slider, W, max 5000, interval 100)
input_datetime.opladning_start (time)
input_datetime.opladning_slut (time)
input_select.vaelg_manuel_opladestyrke:
1400w - 1 fase
1900w - 1 fase
2300w - 1 fase
3000w - 1 fase
3700w - 1 fase
4100w - 3 faser
5500w - 3 faser
6900w - 3 faser
9000w - 3 faser
11000w - 3 faser


DASHBOARD:
type: entities
entities:
  - entity: input_boolean.opladning_afbrudt
  - entity: sensor.sonnenbatterie_225965_production_w
  - entity: sensor.sonnenbatterie_225965_consumption_w
  - entity: sensor.easee_charger_lite_power
  - entity: input_number.energi_buffer_i_watt
  - entity: sensor.easee_charger_lite_status
  - entity: sensor.sonnenbatterie_225965_state_battery_input
    icon: mdi:battery-charging-100
    name: Opladning af batteri
  - entity: sensor.solar_control_excess_solar_energy
  - entity: sensor.excess_solar_energy_30_min_avg
  - entity: input_boolean.prioriter_elbil_opladning
  - entity: sensor.solar_control_excess_solar_energy_2
  - entity: sensor.excess_solar_energy_30_min_avg_2
  - entity: input_boolean.afbrudt_overskuds_opladning
  - entity: input_select.vaelg_manuel_opladestyrke
  - entity: sensor.solcast_pv_forecast_forecast_today
  - entity: sensor.solcast_pv_forecast_forecast_remaining_today
  - entity: sensor.solcast_pv_forecast_forecast_tomorrow
  - entity: input_boolean.oplad_efter_solnedgang
  - entity: input_datetime.opladning_start
  - entity: input_datetime.opladning_slut
title: Dynamisk opladning


For configuration.yaml:

#Statistics
  - platform: statistics
    entity_id: sensor.solar_control_excess_solar_energy
    state_characteristic: median
    max_age:
      minutes: 30
    sampling_size: 1800
    precision: 1
    name: Excess Solar Energy 30 Min Avg
  - platform: statistics
    entity_id: sensor.solar_control_excess_solar_energy_2
    state_characteristic: median
    max_age:
      minutes: 30
    sampling_size: 1800
    precision: 1
    name: Excess Solar Energy 30 Min Avg 2


template:
  - sensor:
      - name: "Solar Control - Excess Solar Energy"
        unique_id: solar_control_excess_solar_energy
        unit_of_measurement: "W"
        icon: mdi:octagram-plus-outline
        state: >
          {# Adjust name for sensor with current solar production in watts #}
          {% set solar_production = states('sensor.sonnenbatterie_225965_production_w') | float %}
          {# Adjust name for sensor with current total electricity consumption in watts #}
          {% set current_consumption = states('sensor.sonnenbatterie_225965_consumption_w') | float %}
          {# Adjust name for sensor with battery charge in watts #}
          {% set battery_charge = states('sensor.sonnenbatterie_225965_state_battery_input') | float %}
          {# Adjust name for sensor with current EV charging in watts, treating 'Unknown' as '0' #}
          {% set ev_charging = states('sensor.easee_charger_lite_power') %}
          {% if ev_charging == 'unknown' %}
            {% set ev_charging = 0 %}
          {% else %}
            {% set ev_charging = ev_charging | float %}
          {% endif %}
          {# Calculation of excess production from solar panels - delete or create buffer helper as needed #}
          {{ solar_production - current_consumption + ev_charging - battery_charge - states('input_number.energi_buffer_i_watt') | float }}

  - sensor:
      - name: "Solar Control - Excess Solar Energy 2"
        unique_id: solar_control_excess_solar_energy_2
        unit_of_measurement: "W"
        icon: mdi:octagram-plus-outline
        state: >
          {# Adjust name for sensor with current solar production in watts #}
          {% set solar_production = states('sensor.sonnenbatterie_225965_production_w') | float %}
          {# Adjust name for sensor with current total electricity consumption in watts #}
          {% set current_consumption = states('sensor.sonnenbatterie_225965_consumption_w') | float %}
          {# Adjust name for sensor with battery charge in watts #}
          {% set battery_charge = states('sensor.sonnenbatterie_225965_state_battery_input') | float %}
          {# Adjust name for sensor with current EV charging in watts, treating 'Unknown' as '0' #}
          {% set ev_charging = states('sensor.easee_charger_lite_power') %}
          {% if ev_charging == 'unknown' %}
            {% set ev_charging = 0 %}
          {% else %}
            {% set ev_charging = ev_charging | float %}
          {% endif %}
          {# Calculation of excess production from solar panels - delete or create buffer helper as needed #}
          {{ solar_production - current_consumption + ev_charging - states('input_number.energi_buffer_i_watt') | float }}
