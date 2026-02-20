# Proposal: Add Energy & Power Units (W, kW, kWh) to ECMA-402

## Status

**Stage**: 0 (seeking champion)

**Champion**: TBD

**Author**: Johan Røed (@johanrd)

## Motivation

Electric vehicles, construction equipment, and building systems report energy consumption in their APIs. An increasing number of web applications are being developed to support the electrification of transport, industry, and the built environment.

It would be helpful if ECMAScript could make developing these applications easier by providing `watt`, `kilowatt` and `kilowatt-hour` as supported units in `Intl.NumberFormat`.

Currently, developers must manually format these units.

**Related issue**: https://github.com/tc39/ecma402/issues/739

## Use Cases

### 1. Energy consumption (kWh)

<table>
  <tr>
    <td valign="top" width="380"><strong>Electricity billing & utility dashboards</strong><br>Smart home monitoring, meter readings</td>
    <td width="180"><a href="https://play.google.com/store/apps/details?id=com.fortum.lmg"><img src="img/1a fortum.png" width="180" alt="Fortum app showing 387 kWh total consumption"></a></td>
  </tr>
  <tr>
    <td valign="top" width="380"><strong>EV charging sessions & battery state</strong><br>Charging apps, fleet management, trip planning<br><sub><a href="https://smartcar.com/docs/api-reference/signals/tractionbattery#param-capacity">Smartcar API</a>, <a href="https://www.fms-standard.com/Truck/down_load/Technical_Specification_rFMS_vehicle_data_V4.0.0_17.09.2021.pdf">rFMS vehicle data spec</a></sub></td>
    <td width="180"><a href="https://play.google.com/store/apps/details?id=com.circlek.ev"><img src="img/1b circlek.png" width="180" alt="Circle K app showing 80 kWh delivered during a charging session"></a></td>
  </tr>
  <tr>
    <td valign="top" width="380"><strong>Solar generation & feed-in</strong><br>Daily/monthly production, grid export</td>
    <td width="180"><a href="https://play.google.com/store/apps/details?id=com.teslamotors.tesla"><img src="img/1c tesla.png" width="180" alt="Tesla app showing 32.8 kWh generated today"></a></td>
  </tr>
</table>

### 2. Power ratings (W, kW)

<table>
  <tr>
    <td valign="top" width="380"><strong>EV charging stations</strong><br>Charging networks, station maps, session details</td>
    <td width="180"><a href="https://chargedrive.com/map"><img src="img/2b chargeanddrive.png" width="180" alt="Chargedrive map showing 150 kW charging station"></a></td>
  </tr>
  <tr>
    <td valign="top" width="380"><strong>Solar & HVAC systems</strong><br>Panel output, inverter dashboards, heating/cooling</td>
    <td width="180"><a href="https://play.google.com/store/apps/details?id=com.teslamotors.tesla"><img src="img/2c tesla.png" width="180" alt="Tesla app showing solar panel kW output and daily kWh generation"></a></td>
  </tr>
  <tr>
    <td valign="top" width="380"><strong>Real-time device power draw</strong><br>Smart plugs, energy meters<br>Appliance ratings, LED lighting, home electronics</td>
    <td width="180"><a href="https://community.home-assistant.io/t/dashboard-real-time-power-meter-with-device-level-detail/420763"><img src="img/2a homeassistant.png" width="180" alt="Home Assistant real-time power meter showing 287.9 W"></a></td>
  </tr>
</table>

### 3. Compound units
- **Energy labelling** — `kilowatt-hour-per-year` (EU energy labels)
- **Carbon accounting** — `gram-per-kilowatt-hour` (CO₂e `g/kWh`)
- **EV efficiency** — `kilowatt-hour-per-kilometer`, `kilowatt-hour-per-mile` (CLDR also defines `kilowatt-hour-per-100-kilometer` as a [unit identifier](https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml))
- **Energy density** — `kilowatt-hour-per-liter` (battery specs)
- **Energy intensity** — `kilowatt-hour-per-kilogram` (industrial production)

## Proposed Solution

Add three sanctioned single unit identifiers to [ECMA-402 §6.6.2 (IsSanctionedSingleUnitIdentifier), Table 2](https://tc39.es/ecma402/#table-sanctioned-single-unit-identifiers):

<table>
  <tr>
   <th colspan="2" align="left">Single Unit Identifier</th>
  </tr>
  <tr>
   <td>watt</td>
   <td>SI derived unit for power (appliance ratings, device specifications)</td>
  </tr>
  <tr>
   <td>kilowatt</td>
   <td>Common power ratings (EV charging rate, solar, HVAC systems)</td>
  </tr>
  <tr>
   <td>kilowatt-hour</td>
   <td>Primary unit for electrical energy consumption (utility bills, EV charging, home energy)</td>
  </tr>
</table>

These three units cover the vast majority of consumer-facing energy applications. ECMA-402 draws from CLDR; these identifiers are already defined and localized.

**AvailableCanonicalUnits()** must include the three new identifiers in its return set.

## API Examples

#### Narrow width, US locale:
```javascript
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour', unitDisplay: 'narrow' }).format(1234.5)
// "1,234.5 kWh"
```

#### Short width, French locale:
```javascript
new Intl.NumberFormat('fr-FR', { style: 'unit', unit: 'kilowatt-hour', unitDisplay: 'short' }).format(1234.5)
// "1 234,5 kWh"
```

#### Long width, German locale:
```javascript
new Intl.NumberFormat('de-DE', { style: 'unit', unit: 'watt', unitDisplay: 'long' }).format(1200)
// "1.200 Watt"
```

#### Power ratings across locales:
```javascript
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt' }).format(5.2)  // "5.2 kW"
new Intl.NumberFormat('fr-FR', { style: 'unit', unit: 'kilowatt' }).format(5.2)  // "5,2 kW"
```

#### Compound units (once base units are sanctioned):
```javascript
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour-per-mile', unitDisplay: 'narrow' }).format(0.29)
// "0.29 kWh/mi"
```

#### Today (before this proposal), these throw RangeError:
```javascript
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour' }).format(42.5)
// RangeError: Invalid unit argument for option unit: kilowatt-hour
```

## Technical Considerations

- **CLDR support**: All proposed units are already defined in Unicode CLDR with complete localization across all supported locales ([unit.xml](https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml))
- **No breaking changes**: Purely additive
- **Minimal payload impact**: Data already exists in CLDR, three units represent a minimal addition.
- **Consistent with existing patterns**: Follows same conventions as `kilogram`/`kilometer`, `fluid-ounce`
- **Userland workarounds and bugs**: Without native support, developers use libraries like [convert-units](https://www.npmjs.com/package/convert-units) (~150K weekly npm downloads), or build custom formatting logic leading to locale bugs ([de-CH kWh bug](https://github.com/home-assistant/frontend/issues/27254), [broken separators](https://community.home-assistant.io/t/decimal-and-thousand-separator-are-not-honored-in-templating/640567))
- **Spec updates**: §6.6.2 IsSanctionedSingleUnitIdentifier (Table 2) and AvailableCanonicalUnits() return set

## Appeal

Energy units have significantly broader utility than existing sanctioned units like `stone` (UK/Ireland body weight), `acre` (land area), or `fluid-ounce` (regional volume measurement), with universal SI standardization and rapidly growing web-facing applications:

- **Electric Vehicles**: Projected to reach 40M annual sales by 2030<sup>[[2]](https://www.iea.org/reports/global-ev-outlook-2024/executive-summary)</sup> (charging networks: ChargePoint 1M+ sessions/month<sup>[[3]](https://www.greencarreports.com/news/1115324_evgo-chargepoint-annual-reports-show-growth-in-electric-car-charging)</sup>, EVgo 1.4M+ drivers<sup>[[4]](https://apps.apple.com/us/app/evgo-fast-ev-charging/id1281660968)</sup>)
- **Building Energy Management**: Mandated by EU EPBD directive<sup>[[1]](https://energy.ec.europa.eu/topics/energy-efficiency/energy-performance-buildings/energy-performance-buildings-directive_en)</sup>
- **Smart Home Market**: $101B in 2024<sup>[[5]](https://www.skyquestt.com/report/smart-home-market)</sup>, projected to reach $226B by 2032 (Home Assistant: 2M+ installations<sup>[[6]](https://www.home-assistant.io/blog/2025/04/16/state-of-the-open-home-recap/)</sup>)
- **Energy Utility Portals**: Major US utilities (PG&E: 16M<sup>[[7]](https://www.pge.com/en/about/company-information/company-profile.html)</sup>, SCE: 15M<sup>[[8]](https://www.sce.com/about-us/who-we-are)</sup> people served) provide kWh-based web dashboards

## Community Support

[Issue #739](https://github.com/tc39/ecma402/issues/739) has received 15 👍 reactions. There is also a thread on [Discourse](https://es.discourse.group/t/addition-of-power-energy-units-to-intl-numberformat/1702).

## CLDR

TC39-TG2 has recommended adding energy unit preference data to [CLDR](https://unicode-org.atlassian.net/browse/CLDR-19201) as the next step for advancing this proposal. Once this data exists in CLDR, the Stage 1 [Smart Unit Preferences](https://github.com/tc39/proposal-smart-unit-preferences) proposal would bring locale-aware automatic unit selection to ECMAScript — including magnitude-based scaling via CLDR's `geq` thresholds.

### Unit Identifier Availability

| Unit Identifier | In CLDR? | Notes |
|----------------|----------|-------|
| `kilowatt-hour` | Yes | Used in `energy/default` preferences |
| `kilowatt` | Yes | Used in `power/engine` preferences |
| `kilowatt-hour-per-100-kilometer` | Yes | Defined as `force-kilowatt-hour-per-100-kilometer` |
| `watt-hour` | No | Would enable full prefix scaling if added. See [CLDR-11454](https://unicode-org.atlassian.net/browse/CLDR-11454) |
| `milliampere-hour` | No | Not currently defined |

## Future Considerations

If `watt-hour` is added to CLDR, it would also enable **battery capacity** formatting (Wh for laptops and portable devices). See [Why watt-hour?](#why-watt-hour)

Additional units could be considered in future proposals if demand emerges:

#### Defined in CLDR:
- `megawatt` - Industrial and power generation facilities
- `gigawatt` - Large-scale power infrastructure
- `milliwatt` - Low-power electronics

#### Not currently in CLDR:
- `watt-hour` - Base unit for energy scaling. See [Why watt-hour?](#why-watt-hour)
- `megawatt-hour` - Utility-scale energy measurement
- `gigawatt-hour` - Utility-scale energy measurement
- `milliampere-hour` – `mAh` used for electronics and batteries

#### Lower priority:
- `joule`, `kilojoule` - Scientific applications (separate domain)
- `british-thermal-unit` - HVAC
- `horsepower` - Automotive (niche web usage)
- `calorie`, `kilocalorie`, `foodcalorie` - Nutrition (separate domain)

**Note**: Energy price comparison (currency/kWh) is a major consumer-facing use case, but ECMA-402 does not currently support compound currency-per-unit formatting.

## FAQ

### Why these three units?

These cover the overwhelming majority of consumer-facing energy applications in web development. Starting focused allows faster adoption, and all three units are already defined with production-quality formatting in CLDR.

### Why watt-hour?

`watt-hour` is not yet defined as an explicit unit in CLDR, so this proposal starts with the three units that have production-quality CLDR support today. If `watt-hour` were added to CLDR, it could be included in ECMA-402 as well — and there are good reasons to consider it:

**It is the natural base unit for energy scaling.** `watt-hour` would enable the full prefix chain — Wh, kWh, MWh, GWh. Real-world applications span this entire range: from Wh for device consumption ([HA thread](https://community.home-assistant.io/t/energy-dashboard-use-wh-instead-of-kwh/724132)) to GWh for grid-scale reporting ([Electricity Maps](https://github.com/electricitymaps/electricitymaps-contrib/issues/4630)). If the [Smart Unit Preferences](https://github.com/tc39/proposal-smart-unit-preferences) proposal (Stage 1) lands, `watt-hour` in CLDR would give energy units automatic magnitude-based scaling — without it, energy would be one of the few common measurement domains without proper scaling support.

**The current compound construction produces suboptimal formatting.** CLDR can construct `watt-hour` from components (`power-watt` + `duration-hour`), but the result is not production-quality:

| Unit | CLDR Status | SHORT format (en) |
|------|-------------|-------------------|
| kilowatt-hour | ✅ Explicit definition | `12.345 kWh` |
| watt-hour | ⚠️ Constructed from components | `12.345 W⋅hr` |

The constructed format uses a separator (`⋅`) and `hr` instead of the natural `Wh`. Other locales have similar issues (German: `W⋅Std.` instead of `Wh`). [CLDR-17881](https://unicode-org.atlassian.net/browse/CLDR-17881) recognizes that commonly-used compound units should have explicit definitions for better formatting.

### Won't this increase bundle sizes?
These units are defined in CLDR with localization across all supported locales. Browsers may need to ship additional translation strings for the three units. However, the per-unit data is small (unit names, abbreviations, and grammatical forms), and three units represent a minimal addition.

## References

- CLDR-19201: https://unicode-org.atlassian.net/browse/CLDR-19201
- Smart Unit Preferences (Stage 1): https://github.com/tc39/proposal-smart-unit-preferences
- Issue #739: https://github.com/tc39/ecma402/issues/739
- Issue #605: https://github.com/tc39/ecma402/issues/605 (related: metric-ton, cubic units)
- CLDR Unit Validity: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml
- CLDR Unit Preferences: https://www.unicode.org/cldr/charts/45/supplemental/unit_preferences.html
- ECMA-402 Spec: https://tc39.es/ecma402/
