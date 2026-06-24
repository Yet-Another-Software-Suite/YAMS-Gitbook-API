# FlyWheelConfig

**Namespace:** `yams::mechanisms::config`
**Java equivalent:** [Java FlyWheelConfig](../../java/config/flywheel-config.md)

`FlyWheelConfig` holds physical and simulation parameters for a `FlyWheel` mechanism.

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
| `WithWheelDiameter(units::meter_t diameter)` | Wheel diameter, used for surface speed calculations |
| `WithWheelMass(units::kilogram_t mass)` | Wheel mass, used to estimate moment of inertia in simulation |
| `WithMOI(units::kilogram_square_meter_t moi)` | Explicit moment of inertia; overrides the mass-based estimate |
| `WithSimColor(const frc::Color8Bit& color)` | Color of the simulated flywheel |

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::meter_t> GetWheelDiameter() const;
std::optional<units::kilogram_t> GetWheelMass() const;
std::optional<units::kilogram_square_meter_t> GetMOI() const;
frc::Color8Bit GetSimColor() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

---

## Example

```cpp
config::FlyWheelConfig flywheelConfig;
flywheelConfig.WithWheelDiameter(0.1016_m)
              .WithWheelMass(0.227_kg)
              .WithTelemetryName("Shooter");
```

{% hint style="info" %}
Provide either `WithWheelMass` or `WithMOI`; providing both causes `WithMOI` to take precedence for simulation. `WithWheelDiameter` is required for surface speed (`meters_per_second`) control targets.
{% endhint %}
