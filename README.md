# Proposal: Add Energy & Power Units (kWh, Wh, kW, W) to ECMA-402

**Stage**: 0 (seeking champion)

**Champion**: TBD

## Status

This proposal adds essential energy and power measurement units to the sanctioned unit identifiers in ECMA-402.

**Related issue**: https://github.com/tc39/ecma402/issues/739

## Motivation

Electric vehicles, smart home devices, solar panels, battery systems, and construction and power industries report energy consumption as kilowatt-hours in their telemetry APIs. An increasing amount of web applications are being developed as tools to help the current electrification of society.

It would be helpful if ECMAScript could make developing these web apps a tiny bit easier by providing kilowatt-hour as a supported unit in `Intl.NumberFormat`.

Currently, developers must manually format these units, losing locale-specific conventions like decimal separators, spacing, and pluralization, or include additional dependencies.

## Use Cases

```javascript
// Today (before this proposal), these throw RangeError:
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour' }).format(42.5)
// RangeError: Invalid unit argument for option unit: kilowatt-hour

// After this proposal - showing width + locale differences:

// Narrow width, US locale
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour', unitDisplay: 'narrow' }).format(1234.5)
// "1,234.5 kWh"

// Short width, French locale (note spacing + comma decimal)
new Intl.NumberFormat('fr-FR', { style: 'unit', unit: 'kilowatt-hour', unitDisplay: 'short' }).format(1234.5)
// "1 234,5 kWh"

// Long width, German locale
new Intl.NumberFormat('de-DE', { style: 'unit', unit: 'watt', unitDisplay: 'long' }).format(1200)
// "1.200 Watt"

// Power ratings across locales
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt' }).format(5.2)  // "5.2 kW"
new Intl.NumberFormat('fr-FR', { style: 'unit', unit: 'kilowatt' }).format(5.2)  // "5,2 kW"

// Compound units (once base units are sanctioned)
new Intl.NumberFormat('en-US', { style: 'unit', unit: 'kilowatt-hour-per-mile', unitDisplay: 'narrow' }).format(0.29)
// "0.29 kWh/mi"
```

## Proposed Solution

Add four sanctioned single unit identifiers to [ECMA-402 §6.6.2 (IsSanctionedSingleUnitIdentifier), Table 2](https://tc39.es/ecma402/#table-sanctioned-single-unit-identifiers):

**Energy Units:**
- `kilowatt-hour` - Primary unit for electrical energy consumption (utility bills, EV charging, home energy)
- `watt-hour` - Smaller energy quantities (battery capacity, device consumption)

**Power Units:**
- `watt` - SI derived unit for power (appliance ratings, device specifications)
- `kilowatt` - Common power ratings (EV charging rate, solar, HVAC systems)

These four units cover the vast majority of consumer-facing energy applications. ECMA-402 draws from CLDR; these identifiers are already defined and localized.

## Specification Changes

**§6.6.2 IsSanctionedSingleUnitIdentifier, Table 2** — append:

| Single Unit Identifier |
|------------------------|
| kilowatt               |
| kilowatt-hour          |
| watt                   |
| watt-hour              |

**AvailableCanonicalUnits()** — must include the four new identifiers in its return set.

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
- **Minimal payload impact**: Data already exists in CLDR, four units represent a minimal addition.
- **Consistent with existing patterns**: Follows same conventions as `kilogram`/`kilometer`, `fluid-ounce`
- **Spec updates**: §6.6.2 IsSanctionedSingleUnitIdentifier (Table 2) and AvailableCanonicalUnits() return set

## Community Support

[Issue #739](https://github.com/tc39/ecma402/issues/739) has received 15 👍 reactions. There is also a thread on [Discourse](https://es.discourse.group/t/addition-of-power-energy-units-to-intl-numberformat/1702).

## Future Considerations

Additional energy and power units exist in CLDR but are excluded from this proposal to maintain focus. They could be considered in future proposals if demand emerges:

**Medium Priority:**
- `megawatt` - Industrial and power generation facilities
- `megawatt-hour` - Utility-scale energy measurement
- `gigawatt` - Large-scale power infrastructure

**Lower Priority:**
- `milliwatt` - Low-power electronics
- `joule`, `kilojoule` - Scientific applications (separate domain)
- `british-thermal-unit` - HVAC 
- `therm-us` - US natural gas billing
- `horsepower` - Automotive (niche web usage)
- `calorie`, `kilocalorie`, `foodcalorie` - Nutrition (separate domain)

## FAQ

### Why these four units?

These cover the overwhelming majority of consumer-facing energy applications in web development. Starting focused allows faster adoption.

### Why kilowatt-hour AND watt-hour?

Both are standard in different contexts:
- `watt-hour`: Small devices ("500 Wh battery")
- `kilowatt-hour`: Consumption ("42 kWh charge", utility bills)

Additionally, having `watt-hour` as a sanctioned single unit allows custom formatters to scale to other prefixes (e.g., MWh) while preserving locale-correct formatting.

### Won't this increase bundle sizes?
These units are defined in CLDR with localization across all supported locales. Browsers may need to ship additional translation strings for the four units. However, the per-unit data is small (unit names, abbreviations, and grammatical forms), and four units represent a minimal addition.

## References

- Issue #739: https://github.com/tc39/ecma402/issues/739
- Issue #605: https://github.com/tc39/ecma402/issues/605 (related: metric-ton, cubic units)
- CLDR Unit Validity: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml
- ECMA-402 Spec: https://tc39.es/ecma402/
