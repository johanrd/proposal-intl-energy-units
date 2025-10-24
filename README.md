# Proposal: Add Energy and Power Units to ECMA-402

**Stage**: 0

**Champion**: TBD

## Status

This proposal adds essential energy and power measurement units to the sanctioned unit identifiers in ECMA-402.

**Related issue**: https://github.com/tc39/ecma402/issues/739

## Motivation

Electric vehicles, smart home devices, solar panels, and battery systems report energy consumption as kilowatt-hours in their telemetry APIs, not to mention the many applications in the construction and power industries. An increasing amount of web applications are going to be developed as tools to help the current electrification of society.

It would be helpful if ecmascript could make developing these web apps a tiny bit easier by providing kilowatt-hour as a supported unit in Intl.NumberFormat.

Currently, developers must manually format these units, losing locale-specific conventions like decimal separators and spacing.

## Use Cases

### Electric Vehicle Charging
```javascript
new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilowatt-hour'
}).format(42.5);
// → "42.5 kWh"
```

### Solar Panel Output
```javascript
new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilowatt'
}).format(5.2);
// → "5.2 kW"
```

### Appliance Power Ratings
```javascript
new Intl.NumberFormat('fr-FR', {
  style: 'unit',
  unit: 'watt'
}).format(1200);
// → "1 200 W"
```

## Proposed Solution

Add four units to Table 2 in ECMA-402 section 6.6.2:

**Energy Units:**
- `kilowatt-hour` - Primary unit for electrical energy consumption (utility bills, EV charging, home energy)
- `watt-hour` - Smaller energy quantities (battery capacity, device consumption)

**Power Units:**
- `watt` - SI base unit for power (appliance ratings, device specifications)
- `kilowatt` - Common power ratings (EV charging rate, solar panels, HVAC systems)

These four units cover the vast majority of consumer-facing energy applications. All units already exist in CLDR with complete localization data.

## Specification Changes

Add to **Table 2: Single units sanctioned for use in ECMAScript** in section 6.6.2:

| Single Unit Identifier |
|------------------------|
| kilowatt               |
| kilowatt-hour          |
| watt                   |
| watt-hour              |

## Prior Art

### CLDR Support
All proposed units are defined in Unicode CLDR with complete localization across all supported locales.

See: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml

### Real-World Usage
- **r[FMS vehicle data specification](https://www.fms-standard.com/Truck/down_load/Technical_Specification_rFMS_vehicle_data_V4.0.0_17.09.2021.pdf)**: Returns energy in watt hours
- **[Tesla API](https://smartcar.com/docs/api-reference/signals/charge#energy-added)**: Returns energy in kilowatt_hours
- **Smart home devices** Report power in W/kW and energy in kWh
- **Solar inverters**  Report generation in W/kW and kWh

## Technical Considerations

- **No breaking changes**: Purely additive
- **Minimal payload impact**: Data already exists in CLDR implementations
- **Consistent with existing patterns**: Follows same conventions as `kilogram`/`kilometer`, `fluid-ounce`

## Community Support

Issue #739 hase received 15 👍 reactions, with support from EV, smart home, and data center industries. There is also a thred on [dascourse](https://es.discourse.group/t/addition-of-power-energy-units-to-intl-numberformat/1702)

## Future Considerations

Additional energy and power units exist in CLDR but are excluded from this proposal to maintain focus. They could be considered in future proposals if demand emerges:

**Medium Priority:**
- `megawatt` - Industrial and power generation facilities
- `megawatt-hour` - Utility-scale energy measurement
- `gigawatt` - Large-scale power infrastructure

**Lower Priority:**
- `milliwatt` - Low-power electronics
- `joule`, `kilojoule` - Scientific applications
- `british-thermal-unit` - HVAC (declining usage)
- `therm-us` - US natural gas billing only
- `horsepower` - Automotive (niche web usage)
- `calorie`, `kilocalorie`, `foodcalorie` - Nutrition (separate domain)

## FAQ

### Why these four units?

These cover the overwhelming majority of consumer-facing energy applications in web development. Starting focused allows faster adoption.

### Why kilowatt-hour AND watt-hour?

Both are standard in different contexts:
- `watt-hour`: Small devices ("500 Wh battery")
- `kilowatt-hour`: Consumption ("42 kWh charge", utility bills)

Additionally, having `watt-hour` as the base unit enables custom formatters to easily create other prefixes (e.g., "3 MWh") by building on the properly defined base unit

### What about compound units?

Compound units (using "-per-") are already supported. Once base units are sanctioned:

```javascript
new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilowatt-hour-per-mile'
}).format(0.29);
// → "0.29 kWh/mi"
```

### Won't this increase bundle sizes?

No significant increase. The localization data already exists in CLDR and is included in browser i18n implementations.

## References

- Issue #739: https://github.com/tc39/ecma402/issues/739
- Issue #605: https://github.com/tc39/ecma402/issues/605 (related: metric-ton, cubic units)
- CLDR Unit Validity: https://github.com/unicode-org/cldr/blob/main/common/validity/unit.xml
- ECMA-402 Spec: https://tc39.es/ecma402/
