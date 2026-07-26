# YUnits

Package: `yams.units`

`YUnits` is a static utility class containing extended unit constants for use with WPILib's `edu.wpi.first.units` system. All values are defined as `Unit` instances compatible with WPILib's `Measure<>` framework.

`YUnits` cannot be instantiated.

---

## Distance Units

| Constant | Symbol | Metric Equivalent |
|----------|--------|------------------|
| `Hands` | — | 0.1016 m (4 inches) |
| `Yards` | yd | 0.9144 m |
| `Cubits` | — | 0.4572 m (18 inches) |
| `Fathoms` | — | 1.8288 m |
| `Chains` | — | 20.1168 m |
| `Furlongs` | — | 201.168 m |
| `Miles` | mi | 1609.344 m |
| `Leagues` | — | 4828.032 m |
| `FootlongSandwich` | — | 0.3048 m |

## Time Units

| Constant | Equivalent |
|----------|-----------|
| `Hours` | 3600 s |
| `Days` | 86400 s |
| `Weeks` | 604800 s |
| `Fortnight` | 1209600 s |
| `Years` | 31536000 s (365 days) |

## Velocity Units

| Constant | Description |
|----------|-------------|
| `MilesPerHour` (MPH) | Imperial speed |
| `FurlongsPerFortnight` (FPF) | A classic unit of slow speed |
| `SandwichPerSecond` | One FootlongSandwich per second |

## Angular Rate Units

| Constant | Description |
|----------|-------------|
| `RotationsPerYear` (RPY) | For very slow mechanisms |
| `RPMPerSecond` | Angular acceleration in rotations-per-minute per second |

## Momentum Units

| Constant | Description |
|----------|-------------|
| `PoundFeetPerSecond` | Linear momentum, imperial |
| `PoundInchesPerSecond` | Linear momentum, smaller scale |
| `PoundFeetSquaredPerSecond` | Angular momentum, imperial |
| `PoundInchesSquaredPerSecond` | Angular momentum, smaller scale |

## Moment of Inertia Units

| Constant | Description |
|----------|-------------|
| `PoundSquareFeet` (lb·ft²) | MOI in imperial units |
| `PoundSquareInches` (lb·in²) | MOI in smaller imperial units |

---

## Usage

```java
import static yams.units.YUnits.*;

Measure<Distance> armLength = Cubits.of(1.5);   // 1.5 cubits ≈ 0.686 m
Measure<Velocity<Distance>> speed = MilesPerHour.of(15);
```

{% hint style="info" %}
`FurlongsPerFortnight` and `FootlongSandwich` are included for completeness (and fun). For actual robot programming, `Meters`, `Inches`, `Feet`, and `Kilograms` from WPILib's standard units are more appropriate.
{% endhint %}
