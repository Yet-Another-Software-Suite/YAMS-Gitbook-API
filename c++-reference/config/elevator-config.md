# ElevatorConfig

**Namespace:** `yams::mechanisms::config`\
**Header:** `<yams/mechanisms/config/ElevatorConfig.hpp>`\
**Java equivalent:** [Java ElevatorConfig](../../java-reference/config/elevator-config.md)

`ElevatorConfig` holds physical and simulation parameters for an `Elevator` mechanism.

***

## Constructor

```cpp
ElevatorConfig() = default;
```

***

## Builder Methods

All methods return `ElevatorConfig&` for chaining.

| Method                                       | Description                                                                    |
| -------------------------------------------- | ------------------------------------------------------------------------------ |
| `WithTelemetryName(const std::string& name)` | Name used for SmartDashboard and log entries                                   |
| `WithMinimumHeight(wpi::units::meter_t height)`   | **Required for simulation.** Minimum allowable carriage height                                              |
| `WithMaximumHeight(wpi::units::meter_t height)`   | **Required for simulation.** Maximum allowable carriage height                                              |
| `WithCarriageMass(wpi::units::kilogram_t mass)`   | **Required for simulation.** Mass of the carriage, used in simulation          |
| `WithIsHorizontal(bool horizontal)`          | When `true`, disables gravity simulation. Useful for horizontal linear sliders |
| `WithSimColor(const wpi::util::Color8Bit& color)`  | Color of the simulated elevator (default: orange)                              |
| `WithAngle(wpi::units::degree_t angle)`           | Angle of the elevator in its movement plane, for Mechanism2d and `GetPose3d()` (default: `90_deg`, vertical) |
| `WithMechanismPositionConfig(const MechanismPositionConfig& config)` | Where the mechanism is on the robot, for `GetPose3d()`, and the plane it moves in. See [MechanismPositionConfig](mechanism-position-config.md) |

> **Note:** Drum or sprocket size is configured on `SmartMotorControllerConfig` via `.WithMechanismCircumference(circumference)` or `.WithMechanismCircumference(gearPitch, teeth)`, and the starting height via `.WithStartingPosition(wpi::units::meter_t)`. In simulation the `Elevator` constructor throws an `ElevatorConfigurationException` if the carriage mass, minimum height, maximum height, starting position or mechanism circumference is missing. Cascading elevator stages are configured via `SmartMotorControllerConfig::WithCascadingElevatorStages(int stages)`.

***

## Getters

```cpp
std::string GetTelemetryName() const;
std::optional<wpi::units::meter_t> GetMinHeight() const;
std::optional<wpi::units::meter_t> GetMaxHeight() const;
std::optional<int> GetStages() const;
const std::vector<wpi::units::meter_t>& GetStageHeights() const;
std::optional<wpi::units::kilogram_t> GetCarriageMass() const;
bool IsHorizontal() const;
wpi::util::Color8Bit GetSimColor() const;
wpi::units::degree_t GetAngle() const;
const MechanismPositionConfig& GetMechanismPositionConfig() const;
```

Getters for optional fields return `std::nullopt` if the corresponding setter was not called.

***

## Exceptions

| Exception                                    | Thrown When                                                                 |
| -------------------------------------------- | --------------------------------------------------------------------------- |
| `exceptions::ElevatorConfigurationException` | Required fields (on `ElevatorConfig` or the motor controller config) are missing when an `Elevator` is constructed in simulation |

***

## Example

```cpp
config::ElevatorConfig elevConfig;
elevConfig.WithMinimumHeight(0.0_m)
          .WithMaximumHeight(1.5_m)
          .WithCarriageMass(4.5_kg)
          .WithTelemetryName("Elevator");
```
