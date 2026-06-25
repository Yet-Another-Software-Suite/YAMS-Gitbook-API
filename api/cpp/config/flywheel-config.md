# FlyWheelConfig

**Namespace:** `yams::mechanisms::config`
**Java equivalent:** [Java FlyWheelConfig](../../java/config/flywheel-config.md)

`FlyWheelConfig` holds physical and simulation parameters for a `FlyWheel` mechanism.

{% hint style="info" %}
**Simulation Note:** Moment of inertia is not configured on `FlyWheelConfig`. Set it via `SmartMotorControllerConfig::WithMOI()`.
{% endhint %}

---

## Constructor

```cpp
FlyWheelConfig() = default;
```

---

## Builder Methods

All methods return `FlyWheelConfig&` for chaining.

| Method | Description |
|--------|-------------|
| `WithTelemetryName(const std::string& name)` | Name used for SmartDashboard and log entries |
| `WithRollerDiameter(units::meter_t diameter)` | Roller/flywheel outer diameter, used for surface speed calculations |
| `WithSimColor(const frc::Color8Bit& color)` | Color of the simulated flywheel (default: orange) |

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::meter_t> GetRollerDiameter() const;
frc::Color8Bit GetSimColor() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

---

## Example

```cpp
config::FlyWheelConfig flywheelConfig;
flywheelConfig.WithRollerDiameter(0.1016_m)
              .WithTelemetryName("Shooter");
```
