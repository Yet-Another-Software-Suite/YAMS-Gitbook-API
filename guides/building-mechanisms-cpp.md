# Building Positional Mechanisms (C++)

**Goal:** Create `Arm`, `Elevator`, and `Pivot` subsystems in C++ with position control and physics simulation.

---

## Prerequisites

Include the main YAMS header or the specific mechanism headers:

```cpp
#include <yams/YAMS.hpp>

// Or individually:
#include <yams/mechanisms/positional/Arm.hpp>
#include <yams/mechanisms/positional/Elevator.hpp>
#include <yams/mechanisms/positional/Pivot.hpp>
#include <yams/mechanisms/config/ArmConfig.hpp>
#include <yams/mechanisms/config/ElevatorConfig.hpp>
#include <yams/mechanisms/config/PivotConfig.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/motorcontrollers/local/SparkWrapper.hpp>
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>
```

---

## Building an Arm

### 1. Configure the `SmartMotorControllerConfig`

Use `WithArmFeedforward` and angular position limits. The motor config must know the gear ratio for accurate position tracking.

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithMotorInverted(false)
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromTeeth(14, 72)))
    .WithFeedback(0.5, 0.0, 0.01)
    .WithArmFeedforward(0.1, 0.5, 0.01, 0.0)   // kS, kV, kA, kG
    .WithMechanismLimits(-90_deg, 90_deg)
    .WithStatorCurrentLimit(40_A)
    .WithMOI(0.6_m, 2.0_kg)                     // arm length, arm mass — required for simulation
    .WithSubsystem(this)
    .WithTelemetry("ShoulderMotor");
```

### 2. Construct the motor controller

```cpp
yams::motorcontrollers::local::SparkWrapper motor{
    new rev::spark::SparkMax(1, rev::spark::SparkLowLevel::MotorType::kBrushless),
    frc::DCMotor::NEO(1),
    motorConfig
};
```

### 3. Create an `ArmConfig`

`WithArmLength` is required for simulation. All other fields are optional.

```cpp
yams::mechanisms::config::ArmConfig armConfig;
armConfig
    .WithArmLength(0.6_m)
    .WithMinAngle(-90_deg)
    .WithMaxAngle(90_deg)
    .WithTelemetryName("Shoulder");
```

### 4. Construct the `Arm`

```cpp
yams::mechanisms::positional::Arm arm{&armConfig, &motor};
```

### 5. Integrate into a subsystem

```cpp
class ShoulderSubsystem : public frc2::SubsystemBase {
public:
    ShoulderSubsystem() {
        motorConfig_
            .WithMotorInverted(false)
            .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
            .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromTeeth(14, 72)))
            .WithFeedback(0.5, 0.0, 0.01)
            .WithArmFeedforward(0.1, 0.5, 0.01, 0.0)
            .WithMechanismLimits(-90_deg, 90_deg)
            .WithStatorCurrentLimit(40_A)
            .WithMOI(0.6_m, 2.0_kg)
            .WithSubsystem(this)
            .WithTelemetry("ShoulderMotor");

        armConfig_
            .WithArmLength(0.6_m)
            .WithMinAngle(-90_deg)
            .WithMaxAngle(90_deg)
            .WithTelemetryName("Shoulder");
    }

    frc2::CommandPtr SetAngleCommand(units::degree_t angle) {
        return arm_.Run(angle);
    }

    frc2::CommandPtr RunToAngle(units::degree_t angle, units::degree_t tolerance = 2_deg) {
        return arm_.RunTo(angle, tolerance);
    }

    frc2::Trigger IsNear(units::degree_t angle, units::degree_t tolerance = 2_deg) {
        return arm_.IsNear(angle, tolerance);
    }

    units::degree_t GetAngle() const { return arm_.GetAngle(); }

    void Periodic() override { arm_.UpdateTelemetry(); }
    void SimulationPeriodic() override { arm_.SimIterate(); }

private:
    yams::motorcontrollers::SmartMotorControllerConfig motorConfig_;
    yams::motorcontrollers::local::SparkWrapper motor_{
        new rev::spark::SparkMax(1, rev::spark::SparkLowLevel::MotorType::kBrushless),
        frc::DCMotor::NEO(1),
        motorConfig_
    };
    yams::mechanisms::config::ArmConfig armConfig_;
    yams::mechanisms::positional::Arm arm_{&armConfig_, &motor_};
};
```

### Command Factories & Triggers

| Method | Returns | Description |
|--------|---------|-------------|
| `Run(degree_t angle)` | `CommandPtr` | Move to angle and hold indefinitely |
| `RunTo(degree_t angle, degree_t tolerance)` | `CommandPtr` | Move to angle, finish when within tolerance |
| `IsNear(degree_t angle, degree_t within)` | `Trigger` | True while within tolerance of angle |
| `Gte(degree_t angle)` | `Trigger` | True when current angle ≥ threshold |
| `Lte(degree_t angle)` | `Trigger` | True when current angle ≤ threshold |
| `Max()` | `Trigger` | True when at or past upper soft limit |
| `Min()` | `Trigger` | True when at or past lower soft limit |

---

## Building an Elevator

### 1. Configure the motor for linear travel

Drum circumference converts rotations to linear distance. Set it on `SmartMotorControllerConfig`.

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(9.0)))
    .WithMechanismCircumference(units::meter_t{2 * std::numbers::pi * 0.025})  // 2π × drum radius
    .WithFeedback(1.2, 0.0, 0.05)
    .WithElevatorFeedforward(0.1, 0.4, 0.02)   // kS, kV, kA
    .WithMeasurementLimits(0.0_m, 1.2_m)
    .WithStatorCurrentLimit(60_A)
    .WithSubsystem(this)
    .WithTelemetry("ElevatorMotor");
```

> **Note:** `WithCascadingElevatorStages(int stages)` is available on `SmartMotorControllerConfig` for multi-stage elevators. Drum radius and stage count do NOT go on `ElevatorConfig`.

### 2. Create an `ElevatorConfig`

`WithCarriageMass` is required for simulation. Height limits are optional but recommended.

```cpp
yams::mechanisms::config::ElevatorConfig elevatorConfig;
elevatorConfig
    .WithCarriageMass(4.5_kg)           // required for simulation
    .WithMinimumHeight(0.0_m)
    .WithMaximumHeight(1.2_m)
    .WithTelemetryName("Elevator");
```

### 3. Construct the `Elevator`

```cpp
yams::mechanisms::positional::Elevator elevator{&elevatorConfig, &motor};
```

### 4. Command Factories & Triggers

| Method | Returns | Description |
|--------|---------|-------------|
| `Run(meter_t height)` | `CommandPtr` | Move to height and hold indefinitely |
| `RunTo(meter_t height, meter_t tolerance)` | `CommandPtr` | Move to height, finish when within tolerance |
| `IsNear(meter_t height, meter_t within)` | `Trigger` | True while within tolerance of height |
| `Gte(meter_t height)` | `Trigger` | True when current height ≥ threshold |
| `Lte(meter_t height)` | `Trigger` | True when current height ≤ threshold |

```cpp
// In periodic:
void Periodic() override { elevator_.UpdateTelemetry(); }
void SimulationPeriodic() override { elevator_.SimIterate(); }
```

---

## Building a Pivot

A `Pivot` is a single-jointed rotational mechanism (similar to `Arm`) without a load-bearing arm length. Use it for wrist joints, turrets, or any rotation that doesn't require an arm-length-based MOI estimate.

### 1. Configure the motor

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(60.0)))
    .WithFeedback(4.0, 0.0, 0.1)
    .WithSimpleFeedforward(0.05, 0.1)
    .WithMechanismLimits(0_deg, 180_deg)
    .WithStatorCurrentLimit(30_A)
    .WithSubsystem(this)
    .WithTelemetry("WristMotor");
```

### 2. Create a `PivotConfig`

```cpp
yams::mechanisms::config::PivotConfig pivotConfig;
pivotConfig
    .WithMinAngle(0_deg)
    .WithMaxAngle(180_deg)
    .WithTelemetryName("Wrist");
```

### 3. Construct the `Pivot`

```cpp
yams::mechanisms::positional::Pivot pivot{&pivotConfig, &motor};
```

### 4. Command Factories & Triggers

Same interface as `Arm` — `Run()`, `RunTo()`, `IsNear()`, `Gte()`, `Lte()`, `Max()`, `Min()`.

```cpp
void Periodic() override { pivot_.UpdateTelemetry(); }
void SimulationPeriodic() override { pivot_.SimIterate(); }
```

---

## Notes

{% hint style="info" %}
`Run(angle)` holds the setpoint indefinitely. Use it for default positions or when the mechanism should stay put until another command interrupts. `RunTo(angle, tolerance)` ends once the mechanism is within tolerance — use it in command sequences to gate the next step.
{% endhint %}

{% hint style="warning" %}
`WithSubsystem(this)` must be set on `SmartMotorControllerConfig` before constructing the motor controller. Without it, command scheduling will not work correctly.
{% endhint %}

---

## Examples

Complete C++ implementations from the `cpptest` reference project:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ArmSubsystem.cpp" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ElevatorSubsystem.cpp" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/TurretSubsystem.cpp" %}

---

## Related Pages

- [Arm (C++)](../api/cpp/mechanisms/arm.md)
- [Elevator (C++)](../api/cpp/mechanisms/elevator.md)
- [Pivot (C++)](../api/cpp/mechanisms/pivot.md)
- [ArmConfig (C++)](../api/cpp/config/arm-config.md)
- [ElevatorConfig (C++)](../api/cpp/config/elevator-config.md)
- [SmartMotorControllerConfig (C++)](../api/cpp/motor-controllers/smart-motor-controller-config.md)
- [Java equivalent: Building an Arm](building-an-arm.md)
- [Java equivalent: Building an Elevator](building-an-elevator.md)
