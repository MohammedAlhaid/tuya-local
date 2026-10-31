# Add Hygger aquarium light

Adds a device definition for a Hygger aquarium LED light.

## Device

| | |
|---|---|
| Manufacturer | Hygger |
| Model | HG028-MS |
| Product id | `2gv5busa` |
| Protocol | Bluetooth, behind a Tuya Bluetooth gateway |

Two units of this light are installed behind the same Tuya Bluetooth gateway
("Community Aquarium Light R" and "Community Aquarium Light L").

The product id is the cloud product id from the Tuya platform.

## DP mapping

LOCAL DPS as reported by the device on a full poll:

```
{"1": true, "3": 600, "5": "0014012a0258", "103": "L8"}
```

The second unit answers the initial status request with a reduced set:

```
{"1": true, "3": 1000, "5": "0014012a0258"}
```

| DP | Type | Use | Observed |
|----|------|-----|----------|
| 1 | boolean | `switch` | `true` |
| 3 | integer | `brightness` (10–1000) | 600, 1000 |
| 5 | hex | `rgbhsv` | `"0014012a0258"` → hue 20°, saturation 298/1000, value 600/1000 |
| 101 | base64 | L8 program raw | not reported |
| 102 | base64 | sunrise/sunset program raw | not reported |
| 103 | string | aquarium mode | `"L8"`, `"M"` |
| 104 | string | aquarium preset | `"030014012a03e8"` |

## Verified

* On/off, brightness and colour (hue/saturation/value) all work, and the
  `rgbhsv` decoding gives the expected colour for the reported value.
* DP 103 `"L8"` is presented as "8 Times Set" and `"M"` as "Manual".
* DP 104 `"030014012a03e8"` is presented as "Fishlight".

## Inferred (from the Tuya app, not observed at runtime)

* The remaining aquarium mode values `"sun"` → "Sunrise/Sunset".
* The remaining aquarium preset values ("Sunlight", "Moonlight", "Ocean Blue",
  "User Defined 1", "User Defined 2").

## Why DPs 101 to 104 are optional

The type detection of the integration uses the DP set of the initial status
response. The second unit answers that request with DPs 1, 3 and 5 only, so
every DP that is not part of that response is marked `optional: true`, as the
contribution guidelines require. The mode and preset selects are populated from
later polls, which is why they show a value once the device is connected.

The light itself is the only part that is fully described by the initial
response, so DPs 1, 3 and 5 are the ones that are always present.

## Why a new definition

The other aquarium light definitions in the repository do not describe this
device:

* `extreme_aquariumlight.yaml` (NemoLight) uses DP 101 as the switch, DP 103 as
  a preview select and DP 104 as a 12/24 hour switch, and adds four colour
  channel lights on DP 105 to 108.
* `arknoah_aquarium_lights.yaml` (Arknoah) uses DP 20 as the switch and DP 101
  to 103 as the brightness of three colour channels.

`duplicates` reports no overlap beyond incidental ones:

```
mellerware_citymove_vacuum matched hygger_aquarium_light 57%
arlec_rf_transmitter matched hygger_aquarium_light 57%
```

For the full poll of the first unit this config is the only match at 100%.

## Limitations

* The raw program DPs 101 and 102 have never returned a value, so those text
  entities stay empty.
* The light is a Bluetooth device behind a gateway, so the integration reaches
  it through the gateway rather than directly.
* Only one model from one manufacturer was tested.
