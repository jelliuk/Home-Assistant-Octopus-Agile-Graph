# Octopus Agile Price Graph for Home Assistant

A compact, theme-aware Home Assistant price graph for **Octopus Agile**, built with ApexCharts.

It shows the current and next electricity price, a rolling 24-hour price graph, negative/cheap/expensive price colouring, and support for **Octoplus Power Down sessions**.

Power Down periods are automatically shown as purple bands on the graph, with additional highlighting when the current or next tariff slot falls inside a joined session.

![Dark mode](screenshots/dark.png)

---

## Features

- 24-hour Octopus Agile electricity price graph
- Current tariff price
- Next tariff price
- Current and next half-hour timestamps
- Live `NOW` marker
- Automatic light/dark theme support
- Dynamic price colouring
- Negative-price highlighting
- Octoplus Power Down session bands
- Automatic `NOW · POWER DOWN` status
- Automatic `NEXT · POWER DOWN` status
- Purple current/next prices when the relevant slot is inside Power Down
- Compact two-column header
- Continuous graph across current-day and next-day Agile data

---

## Requirements

This card uses the following Home Assistant custom cards:

- [ApexCharts Card](https://github.com/RomRider/apexcharts-card)
- [Config Template Card](https://github.com/iantrich/config-template-card)
- [card-mod](https://github.com/thomasloven/lovelace-card-mod)

Installing them through **HACS** is recommended.

You will also need the Octopus Energy integration configured in Home Assistant with entities for:

- Current electricity rate
- Next electricity rate
- Current-day rates
- Next-day rates
- Octoplus Power Down events

The Power Down event entity must expose a `joined_events` attribute containing joined session start/end times.

---

## Installation

### 1. Install the required custom cards

Install the following through HACS:

1. ApexCharts Card
2. Config Template Card
3. card-mod

Restart Home Assistant or reload the frontend if required.

---

### 2. Download the YAML

Copy:

```text
octopus_agile_power_down_graph.yaml
```

into a manual Lovelace card.

You can either paste the full YAML directly into the dashboard editor or store it wherever you normally keep reusable Lovelace configuration.

---

### 3. Find your Octopus entities

In Home Assistant, go to:

```text
Developer Tools → States
```

Find the entities corresponding to:

```text
Current electricity rate
Next electricity rate
Current-day rates
Next-day rates
Octoplus Power Down events
```

The exact entity IDs vary between installations.

---

## Entity configuration

The shared YAML contains placeholder entity IDs.

Search for and replace each of these with the matching entity from your own Home Assistant installation.

### Current rate

```text
sensor.octopus_energy_electricity_your_mpan_your_meter_serial_current_rate
```

Replace with your current-rate sensor.

---

### Next rate

```text
sensor.octopus_energy_electricity_your_mpan_your_meter_serial_next_rate
```

Replace with your next-rate sensor.

---

### Current-day rates

```text
event.octopus_energy_electricity_your_mpan_your_meter_serial_current_day_rates
```

Replace with your current-day rates event entity.

---

### Next-day rates

```text
event.octopus_energy_electricity_your_mpan_your_meter_serial_next_day_rates
```

Replace with your next-day rates event entity.

---

### Octoplus Power Down

```text
event.octopus_energy_your_account_id_octoplus_power_down_events
```

Replace with your Octoplus Power Down events entity.

---

## Example entity mapping

Your entities may look broadly like:

```yaml
sensor.octopus_energy_electricity_xxx_xxx_current_rate
sensor.octopus_energy_electricity_xxx_xxx_next_rate
event.octopus_energy_electricity_xxx_xxx_current_day_rates
event.octopus_energy_electricity_xxx_xxx_next_day_rates
event.octopus_energy_xxx_octoplus_power_down_events
```

Do **not** copy these examples literally. Use the entities from your own Home Assistant installation.

---

# Price colours

The graph uses an Apple-inspired colour palette with separate light and dark theme colours.

The default price scale is:

| Price | Colour |
|---|---|
| Negative | Cyan → Blue |
| 0–5p/kWh | Green |
| 5–20p/kWh | Green → Orange |
| 20–30p/kWh | Orange → Red |
| 30p+/kWh | Red |

The colour transition is gradual rather than using only fixed colour bands.

---

## Negative prices

Negative Agile prices use a cyan-to-blue scale.

Prices close to zero appear cyan, gradually moving toward blue as the price approaches approximately:

```text
-7p/kWh
```

This makes negative periods visually distinct from normal cheap electricity.

---

# Current and next prices

The header displays two values.

Example:

```text
NOW · 17:30                 NEXT · 18:00
56.34 p/kWh                 58.65 p/kWh
```

The displayed times represent the **start time of each half-hour Agile tariff slot**.

---

## Free / negative electricity

If the current price is zero or negative, the current header changes to:

```text
NOW · 17:30 · FREE
```

The price colour continues to follow the negative-price cyan/blue scale.

---

# Octoplus Power Down support

Joined Octoplus Power Down sessions are automatically detected using:

```text
joined_events
```

from the Power Down event entity.

A joined session is displayed as:

- A translucent purple vertical band
- A purple start boundary
- A purple end boundary
- A `POWER DOWN` label above the graph

Example:

```text
                  POWER DOWN
                      ↓
                 │░░░░░░░│
                 │░░░░░░░│
─────────────────│░░░░░░░│────────────
                 │░░░░░░░│
                 │░░░░░░░│
               18:00    19:00
```

---

## Before a Power Down session

If the next half-hour falls inside a joined Power Down session, the next price becomes purple and the header changes to:

```text
NEXT · 18:00 · POWER DOWN
```

For example:

```text
NOW · 17:30             NEXT · 18:00 · POWER DOWN
56.34                   58.65
```

The current price remains in its normal tariff colour because the session has not started yet.

---

## During a Power Down session

When the current time falls inside a joined session:

```text
NOW · 18:00 · POWER DOWN
```

The current price also becomes purple.

If the next half-hour is still inside the same session, both current and next values are purple:

```text
NOW · 18:00 · POWER DOWN     NEXT · 18:30 · POWER DOWN
58.65                        42.13
```

---

## Approaching the end of Power Down

If the current half-hour is inside Power Down but the next half-hour begins outside it:

```text
NOW · 18:30 · POWER DOWN     NEXT · 19:00
42.13                        24.18
```

Only the current value remains purple.

This makes the Power Down state specific to the actual tariff slot rather than simply colouring both values for the entire event.

---

# Power Down colour

Power Down uses a dedicated purple colour:

### Dark mode

```text
#BF5AF2
```

### Light mode

```text
#AF52DE
```

Purple is intentionally separate from the electricity price scale.

This means:

```text
Blue / Cyan / Green / Orange / Red = electricity price
Purple                                = Octoplus Power Down
```

---

## Power Down shading

The default Power Down band opacity is:

```yaml
Dark mode:  0.14
Light mode: 0.09
```

You can change this under:

```yaml
POWER_DOWN_OPACITY:
```

---

# Power Down label position

The Power Down label is raised above the graph so it does not overlap the `NOW` marker.

The default setting is:

```yaml
offsetY: -26
```

To move it higher:

```yaml
offsetY: -32
```

To move it lower:

```yaml
offsetY: -18
```

You can also adjust the horizontal position using:

```yaml
offsetX: 10
```

Positive values move it right and negative values move it left.

---

# NOW marker

The vertical `NOW` marker always retains the colour of the underlying electricity price.

This is intentional.

During Power Down:

- Header colour = Power Down status
- Purple graph band = Power Down period
- NOW line colour = actual electricity price

This allows both pieces of information to remain visible at the same time.

---

# Theme compatibility

The card automatically detects Home Assistant dark mode using:

```javascript
this.hass.themes.darkMode
```

Separate colours are used for:

- Blue
- Cyan
- Green
- Orange
- Red
- Purple

The rest of the card uses Home Assistant theme variables where possible.

No specific dashboard theme is required.

---

# Graph window

The card displays:

```yaml
graph_span: 24h

span:
  start: hour
```

This means the graph begins at the start of the current hour and shows the following 24 hours.

Current-day and next-day Octopus rates are merged to keep the graph continuous across midnight.

---

# Refresh interval

The card uses:

```yaml
update_interval: 1min
```

This keeps:

- The NOW marker
- Current/next slot status
- Power Down state

reasonably current.

There may therefore be a short delay of up to approximately one minute around tariff or Power Down boundaries.

---

# Customisation

## Change graph height

Find:

```yaml
chart:
  height: 190px
```

For a taller graph:

```yaml
height: 230px
```

For a more compact graph:

```yaml
height: 160px
```

---

## Change line thickness

Find:

```yaml
stroke_width: 3
```

and adjust as required.

For example:

```yaml
stroke_width: 2
```

---

## Change the cheap-price threshold

The default card keeps prices below:

```text
5p/kWh
```

green.

Look for:

```javascript
if (price < 5)
```

and the corresponding:

```yaml
- value: 5
```

entries in the colour thresholds.

Make sure to update both the header colour logic and graph thresholds if you change the scale.

---

## Change the expensive-price threshold

The graph reaches full red at:

```text
30p/kWh
```

Look for:

```javascript
if (price < 30)
```

and:

```yaml
- value: 30
```

---

# Troubleshooting

## Card does not appear

Check that all required custom cards are installed:

```text
apexcharts-card
config-template-card
card-mod
```

Then refresh the browser/app.

A hard browser refresh may sometimes be necessary after installing frontend resources.

---

## Entity not found

Verify every placeholder has been replaced with your own entity ID.

Search the YAML for:

```text
your_mpan
your_meter_serial
your_account_id
```

None of these placeholders should remain after configuration.

---

## Price graph is empty

Check the attributes of your current-day and next-day rate event entities.

They should expose rate data containing values similar to:

```yaml
rates:
  - start: ...
    end: ...
    value_inc_vat: ...
```

The card expects:

```text
rates
start
end
value_inc_vat
```

---

## Power Down band does not appear

Check the Power Down event entity in:

```text
Developer Tools → States
```

and inspect its attributes.

The card expects:

```yaml
joined_events:
```

containing entries with:

```text
start
end
```

or:

```text
start
duration_in_minutes
```

The card only displays joined sessions overlapping the visible graph period.

---

## Power Down is listed by Octopus but not shown

Make sure you have actually **joined** the Power Down session.

The graph deliberately uses:

```text
joined_events
```

rather than every available event.

---

## POWER DOWN label overlaps NOW

Adjust:

```yaml
offsetY: -26
```

For example:

```yaml
offsetY: -32
```

---

## NEXT does not turn purple

The card calculates the next Agile half-hour directly from the current time.

For example:

```text
17:35 → 18:00
18:05 → 18:30
18:35 → 19:00
```

The next value only turns purple when that upcoming half-hour overlaps a joined Power Down session.

---

# Contributing

Suggestions, fixes and improvements are welcome.

If you encounter an issue, please include:

- Home Assistant version
- ApexCharts Card version
- Config Template Card version
- Octopus Energy integration version
- Relevant entity attributes

Please remove any personal information before posting entity IDs, logs, screenshots or attributes.

---

# Issues

When opening an issue, please describe:

1. What you expected to happen
2. What actually happened
3. Whether normal Agile prices are working
4. Whether Power Down data exists in `joined_events`
5. Any relevant errors from the browser console or Home Assistant

Please redact your MPAN, meter serial number, account identifiers and any unrelated personal information.

---

# Disclaimer

This is a community Home Assistant configuration.

It is not affiliated with, endorsed by, or maintained by Octopus Energy.

Entity names, attributes and integration behaviour may change between versions of Home Assistant or the Octopus Energy integration.

Always verify electricity pricing and event information using the official Octopus Energy services when accuracy is important.

---

# Licence

This project is available under the MIT Licence.

See:

```text
LICENSE
```

for details.

---

# Credits

Built using:

- Home Assistant
- Octopus Energy integration
- ApexCharts Card
- Config Template Card
- card-mod

If you improve the card or adapt it for another Octopus tariff, contributions are welcome.
