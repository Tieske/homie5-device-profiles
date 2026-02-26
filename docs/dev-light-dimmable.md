# Device: Dimmable Light

This device description describes a dimmable light.

## Dimmable Light introduction

A dimmable-light resembles a light with dimming capabilities and an optional color and temperature (for warm/cold white light).

## Profile name

The profile for this device is `homie-light-dimmable/1/0`. It is located in the device description JSON document in the device level `$profile` attribute, and the `$profile` topic.

## Examples

Description:

```json
{
  "homie": "5.0",
  "name": "main-light",
  "version": 3,
  "$profile": ["homie-light-dimmable/1/0"],
  "nodes": {

    "power": {
      "$profile": ["homie-power-switch/1/0"],
      ...
    },
    "brightness": {
      "$profile": ["homie-level/1/0"],
      ...
    },
    "color": {
      "properties": {
        "value": {
          "settable": true,
          "retained": true,
          "type": "color",
          "format": "rgb"
        }
      }
    },
    "temperature": {
      "$profile": ["homie-level/1/0"],
      ...
    },
    "action": {
      "$profile": ["homie-action/1/0"],
      ...
    },
    "startup": {
      "properties": {
        "value": {
          "settable": true,
          "retained": true,
          "type": "float",
          "format": "1:100",
          "unit": "%"
        },
        "delay": {
          "settable": true,
          "retained": true,
          "type": "float",
          "format": "0:",
          "unit": "s"
        }
      }
    }
  }
}
```

Topics:

```
homie/5/main-light/$profile/homie-light-dimmable/1 → "0"
```

## Device logic

Power and Brightness are independent. The brightness value cannot be 0, as the light would be off. The brightness value must be between 1 and 100. To turn the device off, the power node must be set to `false`.

Some properties have related or overlapping values. The temperature value overlaps with the color value. And if the color is encoded as `hsv` or `rgb`, the brightness value is also included in the color value.

## Nodes

This device-profile implements the following nodes and capability-profiles:

Node-ID | Capability-Profile | required | additional<br/>requirements
-|-|-|-
power | [homie-power-switch/1/0](cap-switch.md#homie-power-switch10) | yes | yes
brightness | [homie-level/1/0](cap-level.md#homie-level10) | yes | yes
action | [homie-action/1/0](cap-action.md#homie-action10) | no | yes
temperature | [homie-level/1/0](cap-level.md#homie-level10) | no | yes
color | n.a. | no | yes
startup | n.a. | no | yes

Additional nodes MAY be added to the device.

### power (required)

The brightness node MUST conform to the following additional requirements:

- A change in state of the `power` node MUST NOT affect the `brightness` value. They are independent.

### brightness (required)

The brightness node MUST conform to the following additional requirements:

- The `unit` attribute of the `value` property MUST be set to `%` (percent).
- The `format` attribute of the `value` property MUST be set to a minimum of `1` and a maximum of `100`.
- `1` represents the lowest possible brightness at which the light is still visibly on.
- A change in `brightness` value MUST NOT affect the `power` state. They are independent.
- Changing the brightness value MUST change the `value` property of the `color` node if it is represented as `hsv` or `rgb` (since they include brightness information).

### action (optional)

The action node (optional) implements the following actions:

action | optional | description
-|-|-
toggle | yes | maps to the `power` node, implementing the `toggle` action of the `homie-power-switch` capability.
step-up | yes | maps to the `brightness` node, implementing the `step-up` action of the `homie-level` capability.
step-down | yes | maps to the `brightness` node, implementing the `step-down` action of the `homie-level` capability.


### temperature (optional)

The temperature node (optional) MUST conform to the following additional requirements:

- The `unit` attribute of the `value` property MUST be set to `K` (Kelvin).
- Changing the temperature value MUST change the `value` property of the `color` node. The `value` property of the `color` node MUST be set to the color matching the temperature and the current brightness.

### color (optional)

The color node (optional) represents the color of the light.

#### Property: value

The `value` property (required) is the property of type `color` that holds the color value and allows to update it.

attributes | value | remark
-|-|-
property-id | "`value`" |
settable | `true` |
retained | `true` |
type | "`color`" |
format | | the standard `format` for colors. It MUST include `rgb`, other color formats MAY be specified in addition.

Upon updating the value, the `brightness` property MUST be updated accordingly if the value is set as an `rgb` or `hsv` value (since they include brightness information).

### Startup (optional)

Dimmable lights might need a minimum brightness level to start. Once started they can be dimmed to a lower brightness, but an initial higher level is required to turn the light on.

The startup node (optional) has 2 properties (required)

property | required |description
-|-|-
value | yes | the minimum brightness level to set when turning the device on
delay | yes | the delay, after which the brightness level can be switched to the brightness level specified by the [brightness](#brightness-required) node.

Logic:

- **WHEN** the device is switched on using the [power](#power-required) node
- **AND** the brightness level as set in the [brightness](#brightness-required) node is **LESS THAN** the startup `value` property
- **THEN** the brightness level should be set to the startup `value` property (without changing the `$target` value in the [brightness](#brightness-required) node)
- **AFTER** `delay` number of seconds the device should set the brightness to the `$target` value as set in the [brightness](#brightness-required) node

#### Property: value

The `value` property (required) is the property of type `float` that holds the start-level brightness value and allows to update it

attributes | value | remark
-|-|-
property-id | "`value`" |
settable | `true` |
retained | `true` |
format | `1:100` | the `format` attribute MUST set minimum (1) and maximum (100), precision MAY be set.
type | "`float`" |
unit | `%` | the unit attribute MUST be `%` to match the brightness property

#### Property: delay

The `delay` property (required) is the property of type `float` that holds the time required to start the light.
Setting the delay to 0 will disable the startup behaviour.

attributes | value | remark
-|-|-
property-id | "`delay`" |
settable | `true` |
retained | `true` |
format | `0:` | the `format` attribute MUST set minimum (0) and MAY set maximum and precision.
type | "`float`" |
unit | `s` | the delay is specified in seconds
