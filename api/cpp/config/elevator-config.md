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
| `WithMinimumHeight(units::meter_t height)` | Minimum allowable carriage height |
| `WithMaximumHeight(units::meter_t height)` | Maximum allowable carriage height |
| `WithCarriageMass(units::kilogram_t mass)` | **Required for simulation.** Mass of the carriage, used in simulation |
| `WithIsHorizontal(bool horizontal)` | When `true`, disables gravity simulation. Useful for horizontal linear sliders |
| `WithSimColor(const frc::Color8Bit& color)` | Color of the simulated elevator (default: orange) |
| `WithAngle(units::degree_t angle)` | Angle of the elevator ligament in Mechanism2d (default: `90_deg`, vertical) |

> **Note:** Drum radius (spool size) is configured on `SmartMotorControllerConfig` via `.WithDrumRadius()` or `.WithMechanismCircumference()`. Cascading elevator stages are configured via `SmartMotorControllerConfig::WithCascadingElevatorStages(int stages)`.

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::meter_t> GetMinHeight() const;
std::optional<units::meter_t> GetMaxHeight() const;
std::optional<units::kilogram_t> GetCarriageMass() const;
bool IsHorizontal() const;
frc::Color8Bit GetSimColor() const;
units::degree_t GetAngle() const;
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
elevConfig.WithMinimumHeight(0.0_m)
          .WithMaximumHeight(1.5_m)
          .WithCarriageMass(4.5_kg)
          .WithTelemetryName("Elevator");
```
