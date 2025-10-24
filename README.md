# Proposal: Add Energy & Power Units (W, kW, kWh) to ECMA-402

## Status

**Stage**: 0 (seeking champion)

**Champion**: TBD

**Author**: Johan Røed (@johanrd)

## Motivation

Electric vehicles and construction equipment have already started to report energy consumption as kilowatt-hours in their telemetry APIs, not to mention the many applications in the building and power industries. An increasing amount of web applications are being developed as tools to help the current electrification of society.

It would be helpful if ECMAScript could make developing these applications a tiny bit easier by providing kilowatt-hour as a supported unit in `Intl.NumberFormat`.

Currently, developers must manually format these units.

**Related issue**: https://github.com/tc39/ecma402/issues/739

## Use Cases

#### Narrow width, US locale:
```javascript
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour', unitDisplay: 'narrow' }).format(1234.5)
// "1,234.5 kWh"
```


#### Short width, French locale
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

## Proposed Solution

Add three sanctioned single unit identifiers to [ECMA-402 §6.6.2 (IsSanctionedSingleUnitIdentifier), Table 2](https://tc39.es/ecma402/#table-sanctioned-single-unit-identifiers):

| Single Unit Identifier |  |
|:------------------------|:-------------|
| kilowatt-hour          | Primary unit for electrical energy consumption (utility bills, EV charging, home energy) |
| kilowatt               | Common power ratings (EV charging rate, solar, HVAC systems) |
| watt                   | SI derived unit for power (appliance ratings, device specifications) |

These three units cover the vast majority of consumer-facing energy applications. ECMA-402 draws from CLDR; these identifiers are already defined and localized.

**AvailableCanonicalUnits()** must include the three new identifiers in its return set.

## Why Not Watt-Hour?

While `watt-hour` would be useful for smaller energy quantities (battery capacity, device consumption), it is **not defined as an explicit unit in CLDR** ([unit.xml](https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml)).

### Current State

CLDR can construct `watt-hour` as a compound unit (`power-watt` + `duration-hour`), but this produces **suboptimal formatting** compared to the explicit `kilowatt-hour`:

| Unit | CLDR Status | SHORT format (en) |
|------|-------------|-------------------|
| kilowatt-hour | ✅ Explicit definition | `12.345 kWh` |
| watt-hour | ⚠️ Constructed from components | `12.345 W⋅hr` |

The constructed format uses a separator (`⋅`) and `hr` instead of the more natural `Wh`. Other locales have similar issues (German: `W⋅Std.` instead of `Wh`).

### CLDR Discussion

[CLDR-11454](https://unicode-org.atlassian.net/browse/CLDR-11454) proposed adding `watt-hour` as an explicit unit but decided it "can be done with compound units" (referring to the construction mechanism). However, [CLDR-17881](https://unicode-org.atlassian.net/browse/CLDR-17881) recognizes that commonly-used compound units should have explicit definitions for better formatting.

### Path Forward

This proposal focuses on the three units with production-quality CLDR support. `watt-hour` and `megawatt-hour` could be added to ECMA-402 once CLDR defines them explicitly with proper formatting.

## Prior Art

### CLDR Support
All proposed units are defined in Unicode CLDR with complete localization across all supported locales.

See: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml

### Real-World Usage
- **[rFMS vehicle data specification](https://www.fms-standard.com/Truck/down_load/Technical_Specification_rFMS_vehicle_data_V4.0.0_17.09.2021.pdf)**: Returns energy in watt hours
- **[Tesla API](https://smartcar.com/docs/api-reference/signals/charge#energy-added)**: Returns energy in kilowatt_hours
- **Smart home devices**: Report power in W/kW and energy in kWh
- **Solar inverters**: Report generation in W/kW and kWh

## Technical Considerations

- **No breaking changes**: Purely additive
- **Minimal payload impact**: Data already exists in CLDR, three units represent a minimal addition.
- **Consistent with existing patterns**: Follows same conventions as `kilogram`/`kilometer`, `fluid-ounce`
- **Userland complexity**: Developers currently rely on unit conversion libraries like [convert-units](https://www.npmjs.com/package/convert-units) (~150K weekly npm downloads for convert-units alone, used by 269+ packages)
- **Spec updates**: §6.6.2 IsSanctionedSingleUnitIdentifier (Table 2) and AvailableCanonicalUnits() return set

## Appeal

Energy units have significantly broader utility than existing sanctioned units like `stone` (UK/Ireland body weight), `acre` (land area), or `fluid-ounce` (regional volume measurement), with universal SI standardization and rapidly growing web-facing applications:

- **Electric Vehicles**: Projected to reach 40M annual sales by 2030<sup>[[2]](https://www.iea.org/reports/global-ev-outlook-2024/executive-summary)</sup> (charging networks: ChargePoint 1M+ sessions/month<sup>[[3]](https://www.greencarreports.com/news/1115324_evgo-chargepoint-annual-reports-show-growth-in-electric-car-charging)</sup>, EVgo 1.4M+ drivers<sup>[[4]](https://apps.apple.com/us/app/evgo-fast-ev-charging/id1281660968)</sup>)
- **Building Energy Management**: Mandated by EU EPBD directive<sup>[[1]](https://energy.ec.europa.eu/topics/energy-efficiency/energy-performance-buildings/energy-performance-buildings-directive_en)</sup>
- **Smart Home Market**: $101B in 2024<sup>[[5]](https://www.skyquestt.com/report/smart-home-market)</sup>, projected to reach $226B by 2032 (Home Assistant: 2M+ installations<sup>[[6]](https://www.home-assistant.io/blog/2025/04/16/state-of-the-open-home-recap/)</sup>)
- **Energy Utility Portals**: Major US utilities (PG&E: 16M<sup>[[7]](https://www.pge.com/en/about/company-information/company-profile.html)</sup>, SCE: 15M<sup>[[8]](https://www.sce.com/about-us/who-we-are)</sup> people served) provide kWh-based web dashboards

## Community Support

[Issue #739](https://github.com/tc39/ecma402/issues/739) has received 15 👍 reactions. There is also a thread on [Discourse](https://es.discourse.group/t/addition-of-power-energy-units-to-intl-numberformat/1702).

## Future Considerations

Additional energy and power units exist in CLDR but are excluded from this proposal to maintain focus. They could be considered in future proposals if demand emerges:

**Medium Priority (defined in CLDR):**
- `megawatt` - Industrial and power generation facilities
- `gigawatt` - Large-scale power infrastructure
- `milliwatt` - Low-power electronics

**Not Currently in CLDR (would require upstream CLDR work first):**
- `watt-hour` - Smaller energy quantities (battery capacity, device consumption)
- `megawatt-hour` - Utility-scale energy measurement

**Lower Priority:**
- `joule`, `kilojoule` - Scientific applications (separate domain)
- `british-thermal-unit` - HVAC
- `therm-us` - US natural gas billing
- `horsepower` - Automotive (niche web usage)
- `calorie`, `kilocalorie`, `foodcalorie` - Nutrition (separate domain)

## FAQ

### Why these three units?

These cover the overwhelming majority of consumer-facing energy applications in web development. Starting focused allows faster adoption, and all three units are already defined with production-quality formatting in CLDR.

### What about smaller units like watt-hour?

See the [Why Not Watt-Hour?](#why-not-watt-hour) section above. While useful, `watt-hour` is not currently defined as an explicit unit in CLDR and produces suboptimal formatting when constructed as a compound unit.

### Won't this increase bundle sizes?
These units are defined in CLDR with localization across all supported locales. Browsers may need to ship additional translation strings for the three units. However, the per-unit data is small (unit names, abbreviations, and grammatical forms), and three units represent a minimal addition.

## References

- Issue #739: https://github.com/tc39/ecma402/issues/739
- Issue #605: https://github.com/tc39/ecma402/issues/605 (related: metric-ton, cubic units)
- CLDR Unit Validity: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml
- ECMA-402 Spec: https://tc39.es/ecma402/
