# Power Flow Card

[![hacs_badge](https://img.shields.io/badge/HACS-Default-41BDF5.svg?style=flat-square)](https://github.com/hacs/integration)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/ulic75/power-flow-card?style=flat-square)
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/ulic75/power-flow-card/CI?style=flat-square)
![GitHub all releases](https://img.shields.io/github/downloads/ulic75/power-flow-card/total?style=flat-square)
[![ko-fi support](https://img.shields.io/badge/support-me-ff5e5b?style=flat-square&logo=ko-fi)](https://ko-fi.com/ulic75)

## Goal/Scope

Display current power, gas, and water usage in a display that matches the the official Energy Distribution card included with [Home Assistant](https://home-assistant.io/) as much as possible. As such additional features that fall outside of that scope will not be included.

![screenshot](https://user-images.githubusercontent.com/5641964/202307818-4fb8923e-7fea-451f-bb5a-c840cfcc2dae.png)

## Install

### HACS (recommended)

This card is available in [HACS](https://hacs.xyz/) (Home Assistant Community Store).
<small>_HACS is a third party community store and is not included in Home Assistant out of the box._</small>

### Manual install

1. Download and copy `power-flow-card.js` from the [latest release](https://github.com/ulic75/power-flow-card/releases/latest) into your `config/www` directory.

2. Add the resource reference as decribed below.

### Add resource reference

If you configure Dashboards via YAML, add a reference to `power-flow-card.js` inside your `configuration.yaml`:

```yaml
resources:
  - url: /local/power-flow-card.js
    type: module
```

Else, if you prefer the graphical editor, use the menu to add the resource:

1. Make sure, advanced mode is enabled in your user profile (click on your user name to get there)
2. Navigate to Settings -> Dashboards
3. Click three dot icon
4. Select Resources
5. Hit (+ ADD RESOURCE) icon
6. Enter URL `/local/power-flow-card.js` and select type "JavaScript Module".
   (Use `/hacsfiles/power-flow-card/power-flow-card.js` and select "JavaScript Module" for HACS install if HACS didn't do it already)

## Using the card

I recommend looking at the [Example usage section](#example-usage) to understand the basics to configure this card.
(also) pay attention to the **required** options mentioned below.

### Options

#### Card options

| Name              | Type     |   Default    | Description                                                                                                                                                                  |
| ----------------- | -------- | :----------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type              | `string` | **required** | `custom:power-flow-card`.                                                                                                                                                    |
| entities          | `object` | **required** | One or more sensor entities, see [entities object](#entities-object) for additional entity options.                                                                          |
| title             | `string` |              | Shows a title at the top of the card.                                                                                                                                        |
| dashboard_link    | `string` |              | Shows a link to an Energy Dashboard. Should be a url path to location of your choice. If you wanted to link to the built-in dashboard you would enter `/energy` for example. |
| inverted_entities | `string` |              | Comma seperated list of entities that should be inverted (negative for consumption and positive for production). See [example usage](#inverted-entities-example).            |
| kw_decimals       | `number` |      1       | Number of decimals rounded to when kilowatts are displayed.                                                                                                                  |
| w_decimals        | `number` |      1       | Number of decimals rounded to when watts are displayed.                                                                                                                      |
| min_flow_rate     | `number` |     .75      | Represents the fastest amount of time in seconds for a flow dot to travel from one end to the other, see [flow formula](#flow-formula).                                      |
| max_flow_rate     | `number` |      6       | Represents the slowest amount of time in seconds for a flow dot to travel from one end to the other, see [flow formula](#flow-formula).                                      |
| watt_threshold    | `number` |      0       | The number of watts to display before converting to and displaying kilowatts. Setting of 0 will always display in kilowatts.                                                 |

#### Entities object

At least one of _grid_, _battery_, or _solar_ is required. All entites (except _battery_charge_) should have a `unit_of_measurement` attribute of W(watts) or kW(kilowatts).

| Name           | Type                | Description                                                                                                                                                                                                     |
| -------------- | :------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| grid           | `string` / `object` | Entity ID of a sensor supporting a single state with negative values for production and positive values for consumption or an object for [split entites](#split-entities). Examples of both can be found below. |
| battery        | `string` / `object` | Entity ID of a sensor supporting a single state with negative values for production and positive values for consumption or an object for [split entites](#split-entities). Examples of both can be found below. |
| battery_charge | `string`            | Entity ID providing a state with the current percentage of charge on the battery.                                                                                                                               |
| solar          | `string`            | Entity ID providing a state with the value of generation.                                                                                                                                                       |
| gas            | `string`            | Entity ID providing a state value of consumption                                                                                                                                                                |
| water          | `string`            | Entity ID providing a state value of consumption                                                                                                                                                                |

#### Split entities

Can be use with either Grid or Battery configuration. The same `unit_of_measurement` rule as above applies.

| Name        | Type     | Description                                                                                       |
| ----------- | -------- | ------------------------------------------------------------------------------------------------- |
| consumption | `string` | Entity ID providing a state value for consumption, this is required if using a split grid object. |
| production  | `string` | Entity ID providing a state value for production                                                  |

### Example usage

#### Combined Entites Example

Using combined entities for grid, battery and solor that support positive state values for consumption and negative state values for production.

```yaml
type: custom:power-flow-card
entities:
  battery: sensor.battery_in_out
  battery_charge: sensor.battery_percent
  grid: sensor.grid_in_out
  solar: sensor.solar_out
```

#### Inverted Entities Example

Using combined entites as above but where the battery and grid entities are inverted (negative = consumption and positive = production).

```yaml
type: custom:power-flow-card
entities:
  battery: sensor.battery_in_out
  battery_charge: sensor.battery_percent
  grid: sensor.grid_in_out
  solar: sensor.solar_out
inverted_entities: battery, grid
```

#### Split Entites Example

Using split entities for grid and battery where each consumption and production entity state has a positive value.

```yaml
type: custom:power-flow-card
entities:
  battery:
    consumption: sensor.battery_out
    production: sensor.battery_in
  battery_charge: sensor.battery_percent
  grid:
    consumption: sensor.grid_out
    production: sensor.grid_in
  solar: sensor.solar_out
```

### Flow Formula

This formula is based on the offical formula used by the Energy Distribution card.

```js
max - (value / totalLines) * (max - min);
// max = max_flow_rate
// min = min_flow_rate
// value = line value, solar to grid for example
// totalLines = gridConsumption + solarConsumption + solarToBattery +
//   solarToGrid + batteryConsumption + batteryFromGrid + batteryToGrid
```

I'm not 100% happy with this. I'd prefer to see the dots travel slower when flow is low, but faster when flow is high. For example if the only flow is Grid to Home, I'd like to see the dot move faster if the flow is 15kW, but slower if it's only 2kW. Right now the speed would be the same. If you have a formula you'd like to propose please submit a PR.

## Credits

- [HA Energy Distribution Card](https://www.home-assistant.io//dashboards/energy/#energy-distribution)
- [@angular/cdk](https://github.com/angular/components/tree/main/src/cdk)
- [Jack Moore](https://www.jacklmoore.com/notes/rounding-in-javascript/)
