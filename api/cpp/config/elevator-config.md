# ElevatorConfig

**Namespace:** `yams::mechanisms::config`
**Java equivalent:** [Java ElevatorConfig](../../java/config/elevator-config.md)

`ElevatorConfig` holds physical and simulation parameters for an `Elevator` mechanism.

---

## Constructor

```cpp
ElevatorConfig() = default;
```

---

## Builder Methods

All methods return `ElevatorConfig&` for chaining.

| Method | Description |
|--------|-------------|
| `WithTelemetryName(const std::string& name)` | Name used for SmartDashboard and log entries |
| `WithMinHeight(units::meter_t height)` | Minimum allowable carriage height |
| `WithMaxHeight(units::meter_t height)` | Maximum allowable carriage height |
| `WithCarriageMass(units::kilogram_t mass)` | Mass of the carriage, used in simulation |
| `WithDrumRadius(units::meter_t radius)` | Radius of the driving drum or spool |
| `WithElevatorAngle(units::degree_t angle)` | Angle of travel from horizontal (default: `90_deg`, vertical) |
| `WithNumStages(int stages)` | Number of cascaded stages; multiplies effective travel |
| `WithSimColor(const frc::Color8Bit& color)` | Color of the simulated elevator |

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::meter_t> GetMinHeight() const;
std::optional<units::meter_t> GetMaxHeight() const;
std::optional<units::kilogram_t> GetCarriageMass() const;
std::optional<units::meter_t> GetDrumRadius() const;
std::optional<units::degree_t> GetElevatorAngle() const;
std::optional<int> GetNumStages() const;
frc::Color8Bit GetSimColor() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

---

## Exceptions

| Exception | Thrown When |
|-----------|-------------|
| `exceptions::ElevatorConfigurationException` | Required fields are missing or invalid when used to construct an `Elevator` |

---

## Example

```cpp
config::ElevatorConfig elevConfig;
elevConfig.WithMinHeight(0.0_m)
          .WithMaxHeight(1.5_m)
          .WithCarriageMass(4.5_kg)
          .WithDrumRadius(0.0254_m)
          .WithNumStages(2)
          .WithTelemetryName("Elevator");
```
