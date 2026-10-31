# Add Carrier TC86 A air conditioner

Adds a device definition for a Carrier air conditioner that is controlled
locally over the LAN.

## Device

| | |
|---|---|
| Manufacturer | Carrier |
| Model | TC86 A |
| Product id | `qxjcbkkyqdtuyobd` |
| Protocol | WiFi (local) |

Three identical units were tested: "Master Bedroom AC Local", "Living Room AC N"
and "Living Room AC S". All three report the same product id.

The product id is the cloud product id from the Tuya platform. It is also the
id that the integration reports for these units during local discovery (the
integration logs the discovered product id when it is not listed in the config
of the device it belongs to), so both sources agree.

## DP mapping

LOCAL DPS as reported by the device on a full poll:

```
{"1": true, "2": 24, "3": 24, "4": "Cool", "5": "1", "6": "2", "22": 0, "101": "NOerror", "103": 24}
```

| DP | Type | Use | Observed |
|----|------|-----|----------|
| 1 | boolean | `hvac_mode` | `true` / `false` (on/off); the actual mode comes from DP 4 |
| 2 | integer | `temperature` (set point) | 24, 25 |
| 3 | integer | `current_temperature` | 24 |
| 4 | string | `mode` (hidden, used as the constraint for DP 1) | `"Cool"` |
| 5 | string | `fan_mode` | `"1"` |
| 6 | string | *not mapped* | `"2"` |
| 22 | bitfield | fault bitmap | `0` (healthy) |
| 101 | string | fault text | `"NOerror"` |
| 103 | integer | *not mapped* (repeats the current temperature) | 24 |

The fault bitmap and the fault text are reported by the device but hidden in the
Tuya app, so they are exposed as a `problem` binary sensor with `fault_code` and
`fault_text` attributes.

## Verified

* Power and mode: the climate entity reports `cool` for DP 1 `true` with DP 4
  `"Cool"`, and `off` for DP 1 `false`.
* Current temperature (DP 3) and set point (DP 2) are read and written as
  expected.
* Fan speed: DP 5 `"1"` maps to `low`.
* Faults: DP 22 `0` reports the problem sensor as off, and DP 101 is exposed as
  the fault text.

## Inferred (from the Tuya app, not observed at runtime)

* The remaining mode strings `"Heat"`, `"Dry"` and `"Fan"`.
* The remaining fan speeds `"2"`/`"3"`/`"4"` → medium/high/auto.
* The 20–30 °C set point range.
* The meaning of the individual bits in the DP 22 fault bitmap (the app labels
  them E1–E6).

## Not exposed

* DP 6 and DP 103 are not mapped; their purpose is not known.

## Why a new definition

`best_match` for the observed DP set returns this config as the only match
(78%). The closest bundled generic air conditioner, `beltax_aircondition.yaml`,
cannot be used for this device: it declares DP 101 as a boolean `preset_mode`
and DP 103 as a string `heat_supported`, while this device reports DP 101 as a
string and DP 103 as an integer, so the type check in `matches()` rejects it.
Changing those two DPs in that file would change the behaviour of the products
already listed there (Beltax, Medion, Monzana).

`duplicates` reports the following overlaps, none of which is a duplicate:

```
haier_airmart_ac matched carrier_tc86a_airconditioner 71%
feit_dimmer matched carrier_tc86a_airconditioner 71%
ideal_clima_fancoil matched carrier_tc86a_airconditioner 71%
```

They differ in the DP set: this device reports a fault bitmap on DP 22 with a
fault text on DP 101 and a hidden mode DP 4, which those configs do not have.

## Limitations

* Only one model from one manufacturer was tested.
* The model string is the label on the unit as read by the submitter; it has not
  been confirmed against the Tuya cloud product record.
* No fault has ever been raised on any of the three units, so the fault bitmap
  has only been observed in its healthy state.
