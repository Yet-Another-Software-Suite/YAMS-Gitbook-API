# ArmConfig

**Namespace:** `yams::mechanisms::config`
**Java equivalent:** [Java ArmConfig](../../java/config/arm-config.md)

`ArmConfig` holds physical and simulation parameters for an `Arm` mechanism.

---

## Constructor

```cpp
ArmConfig() = default;
```

---

## Builder Methods

All methods return `ArmConfig&` for chaining.

> **Required for simulation:** `WithArmLength()` must be set for `SingleJointedArmSim` to work correctly. All other methods are optional.

| Method | Required | Description |
|--------|----------|-------------|
| `WithArmLength(units::meter_t length)` | **Required for simulation** | Arm length from pivot to end effector. Without this, the physics simulation cannot model the arm. |
| `WithMinAngle(units::degree_t angle)` | Optional | Minimum allowable mechanism angle |
| `WithMaxAngle(units::degree_t angle)` | Optional | Maximum allowable mechanism angle |
| `WithTelemetryName(const std::string& name)` | Optional | Name used for SmartDashboard and log entries |
| `WithSimColor(const frc::Color8Bit& color)` | Optional | Color of the simulated arm ligament |

---

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<units::degree_t> GetMinAngle() const;
std::optional<units::degree_t> GetMaxAngle() const;
std::optional<units::meter_t> GetArmLength() const;
frc::Color8Bit GetSimColor() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

---

## Exceptions

| Exception | Thrown When |
|-----------|-------------|
| `exceptions::ArmConfigurationException` | Required fields are missing or values are invalid when used to construct an `Arm` |

---

## Example

```cpp
config::ArmConfig armConfig;
armConfig.WithMinAngle(-90_deg)
         .WithMaxAngle(90_deg)
         .WithArmLength(0.6_m)
         .WithTelemetryName("Arm");
```
