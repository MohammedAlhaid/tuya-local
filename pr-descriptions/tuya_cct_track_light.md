# Add CCT track light

Adds a device definition for a Tuya based CCT (white / colour temperature only)
LED track light.

## Device

| | |
|---|---|
| Manufacturer | not known (OEM models, see below) |
| Models | HU-S3D0101-10, ME-S35-ZD106-10, HU-S3FD0601-10, HU-S3D0101-15 |
| Product ids | `pdgzvgfu`, `cnfc04xs`, `tumfymby` |
| Protocol | WiFi (local) |

Ten identical units were tested (a living room track light with five fittings,
each with two segments). All ten report the same DP set and the same values.

The product ids are the cloud product ids from the Tuya platform. The brand of
these OEM models is not known, so only the model is listed for each product.

## DP mapping

LOCAL DPS as reported by the device on a full poll:

```
{"1": false, "2": "white", "3": 163, "4": 0, "7": 0}
```

| DP | Type | Use | Observed |
|----|------|-----|----------|
| 1 | boolean | `switch` | `false` |
| 2 | string | *not mapped* (reports the colour mode, `"white"`) | `"white"` |
| 3 | integer | `brightness` (10–1000) | 163 |
| 4 | integer | `color_temp` (0–1000, mapped to 2700–6500 K) | 0 |
| 6 | string | scene (not reported by the device) | – |
| 7 | integer | timer (0–86400 s) | 0 |
| 34 | boolean | do not disturb (not reported by the device) | – |

## Verified

* On/off, brightness and colour temperature all work.
* The colour temperature is sent as 0–1000 by the device and is mapped to
  2700–6500 K, so DP 4 value `0` is presented as 2700 K.
* The light reports only `color_temp` in `supported_color_modes` – it has no
  RGB channel, so no `hs` mode is offered.
* The timer DP 7 is present on the device and is offered as a timer entity.

## Inferred (from the Tuya app, not observed at runtime)

* The four scene payloads. They are the four white/brightness scenes of the
  generic light profile; the four remaining payloads of that profile carry RGB
  data that this CCT only fitting cannot render, and the app does not offer
  them for this device.
* That DP 6 and DP 34 are write only. They are not part of any poll, so the
  scene select stays `unknown` and the do not disturb switch is unavailable
  until the device reports them.

## Why a new definition

The generic `zigbee_smart_bulb.yaml` profile matches this device on every DP it
uses, and `duplicates` therefore reports it as a 100% match for this file. That
profile cannot be used as is:

* it declares DP 5 `rgbhsv` and a DP 2 colour mode that maps `colour` to `hs`,
  which would advertise an `hs` colour mode that this CCT only fitting cannot
  render (it reports no DP 5 at all),
* it declares DP 33 power memory, which this device does not report,
* it maps four additional scene payloads that carry RGB data.

Removing those from `zigbee_smart_bulb.yaml` would change the behaviour of the
products already listed there (Moes, Peteme, Kojima), so a separate definition
is used instead. The only other overlap is `olight_sphere_light` (100%), which
is a decorative RGB light with a large effect list and is not applicable.

## Limitations

* DP 2 is not exposed as a colour mode, so the attribute is not shown.
* The scene select never reports a value, so it can be written but not read
  back.
* The do not disturb switch has never been observed on the device.
* The brand of the fittings is unknown; if it should be added to the products
  entries, please say which one.
* The three product ids cover four model names; the individual units that were
  tested could not be mapped to a specific model, they all report the same
  DPs.
