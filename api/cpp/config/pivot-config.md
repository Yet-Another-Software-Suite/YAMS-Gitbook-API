# PivotConfig

**Namespace:** `yams::mechanisms::config`
**Java equivalent:** [Java PivotConfig](../../java/config/pivot-config.md)

`PivotConfig` holds physical and simulation parameters for a `Pivot` mechanism. A pivot is a single-jointed rotational mechanism without a load-bearing length (use `ArmConfig` if arm length affects simulation accuracy).

---

## Constructor

```cpp
PivotConfig() = default;
```

---

## Builder Methods

All methods return `PivotConfig&` for chaining.

| Method | Description |
|--------|-------------|
| `WithTelemetryName(const std::string& name)` | Name used for SmartDashboard and log entries |
| `WithMinAngle(units::degree_t angle)` | Minimum allowable mechanism angle |
| `WithMaxAngle(units::degree_t angle)` | Maximum allowable mechanism angle |
| `WithSimColor(const frc::Color8Bit& color)` | Color of the simulated pivot ligament |

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::degree_t> GetMinAngle() const;
std::optional<units::degree_t> GetMaxAngle() const;
frc::Color8Bit GetSimColor() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

---

## Exceptions

| Exception | Thrown When |
|-----------|-------------|
| `exceptions::PivotConfigurationException` | Required fields are missing or invalid when used to construct a `Pivot` |

---

## Example

```cpp
config::PivotConfig pivotConfig;
pivotConfig.WithMinAngle(0_deg)
           .WithMaxAngle(180_deg)
           .WithTelemetryName("Wrist");
```
