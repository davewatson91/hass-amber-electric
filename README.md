# Home Assistant Plugin for Amber Electric Real-Time Pricing

## ** ATTENTION **

## I have been in contact with Amber -> this API will be deprecated over a 6 month period, with a new API almost ready for release. One of the developers at Amber (creating the API, will also be developing (in his personal capacity I believe) an integration for Home Assistant. 



#### This fork is based on the repo originally developed by lewisbenge/hass-amber-electric, continued here for ongoing maintenance due to a group of users relying on this for the Australian market. Please note I am not a qualified dev, and welcome any assistance to keeping this maintained. 

A custom component for Home Assistant (www.home-assistant.io) to pull the latest energy prices from the Amber Electric (https://www.amberelectric.com.au/) REST API based on Post Code.

# How to install

Copy custom_components/amberelectric to your hass data directory (where your configuration.yaml lives). It should go into the same directory structure (`YOUR_CONFIG_DIRECTORY/custom_components/amberelectric`)

Add a sensor into your configuration.yaml file that defines the Post Code you want to lookup.

```yaml
sensor:
  - platform: amberelectric
    postcode: "2000"
```

After editing your configuration.yaml, you will need to restart Home Assistant. 

Once deployed, you should see 3 sensors added to your Home Assistant entities list, one reflects the current solar feed-in tarrif, and the second is the real-time grid price charged by Amber (including their network fees), and a third for real-time controlled grid pricing.

# Example Usage

## Analysing Price Predictions

The 12 hour price predictions from Amber for both solar FiT and grid usage are available as an attribute of the sensor. To use this data-set in automation or scenes, it is best to return the array using a template to achieve what you need.

The following example uses a template to return the prices based on most expense:

```python
{% for price_forecast in states.sensor.amber_general_usage_price.attributes["price_forcecast"] | sort(attribute='price') | reverse %}
{{price_forecast['pricing_period']}}: {{price_forecast['price']}}
{%- endfor -%}
```

To get a mean value for the next 12 hours (so you can look at price change to determine increases), you can leverage a template such as this:

```python
{% set mean__price_value = (states.sensor.amber_general_usage_price.attributes["price_forcecast"] | sum(attribute='price')) /states.sensor.amber_general_usage_price.attributes["price_forcecast"] | length()   %}
{{mean_price_value}}
```

Typically within a template you want to return a single value, to this example would need to be tailored to your specific use-case. You can experiment with Templates in the Developer Tools section of your Home Assistant portal.

## Example sensor template

```sensor:
  - platform: amberelectric
    postcode: "2000"
    
  - platform: template
    sensors:
      amber_peak_predicted_2h:
        friendly_name: "Amber 2 hour peak predicted"
        unit_of_measurement: "c/kWh"
        value_template: >-
          {% set forecast = states.sensor.amber_general_usage_price.attributes['price_forcecast'] %}
          {% set highest = forecast[0:4] | sort(reverse=true, attribute='price') | first() %}
          {{highest['price']}}
      amber_peak_predicted_4h:
        friendly_name: "Amber 4 hour peak predicted"
        unit_of_measurement: "c/kWh"
        value_template: >-
          {% set forecast = states.sensor.amber_general_usage_price.attributes['price_forcecast'] %}
          {% set highest = forecast[0:8] | sort(reverse=true, attribute='price') | first() %}
          {{highest['price']}}
```
# Troubleshooting

## Numbers don't look right?

Some postcodes have multiple power distributors, and the API will return the primary one. If you are on the secondary one, your numbers may be out. To do that, you can change the network name in the config
sensor:
```yaml
- platform: amberelectric
  postcode: '3000'
  network_name: 'Jemena'
```

The [Australian Energy Regulator](https://www.aer.gov.au/consumers/who-is-my-distributor) has more information about finding out who your distributor is.

# Ideas

This project is early in it's creation. If you have ideas on how you want to use the Amber real-time data inside Home Assistant please let me know and I'll try my best to update to code to reflect the various use-cases.
