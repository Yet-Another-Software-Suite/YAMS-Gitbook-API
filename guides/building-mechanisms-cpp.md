# Building Positional Mechanisms (C++)

**Goal:** Create `Arm`, `Elevator`, and `Pivot` subsystems in C++ with position control and physics simulation.

***

## Prerequisites

Install the Commands v2 YAMS vendordep (`https://cdn.yassrobotics.com/yams_commands2.json`); C++ is not part of the Commands v3 vendordep. Then include the main YAMS header or the specific mechanism headers. `YAMS.hpp` does not pull in the vendor wrappers, so include the wrapper for your motor controller explicitly.

```cpp
#include <yams/YAMS.hpp>

// Or individually:
#include <yams/mechanisms/positional/Arm.hpp>
#include <yams/mechanisms/positional/Elevator.hpp>
#include <yams/mechanisms/positional/Pivot.hpp>
#include <yams/mechanisms/config/ArmConfig.hpp>
#include <yams/mechanisms/config/ElevatorConfig.hpp>
#include <yams/mechanisms/config/PivotConfig.hpp>
#include <yams/gearing/GearBox.hpp>
#include <yams/gearing/MechanismGearing.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>

// Vendor wrappers (always included individually):
#include <yams/motorcontrollers/local/SparkWrapper.hpp>
#include <yams/motorcontrollers/local/NovaWrapper.hpp>
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>
#include <yams/motorcontrollers/remote/TalonFXSWrapper.hpp>
```

WPILib 2027 C++ types live in the `wpi::` namespaces (`wpi::units`, `wpi::math`, `wpi::cmd`) with headers under `<wpi/...>`:

```cpp
#include <wpi/commands2/CommandPtr.hpp>
#include <wpi/commands2/SubsystemBase.hpp>
#include <wpi/commands2/button/Trigger.hpp>
#include <wpi/hardware/bus/CANPort.hpp>
#include <wpi/math/controller/ArmFeedforward.hpp>
#include <wpi/math/controller/ElevatorFeedforward.hpp>
#include <wpi/math/controller/SimpleMotorFeedforward.hpp>
#include <wpi/math/system/DCMotor.hpp>
#include <wpi/units/angle.hpp>
#include <wpi/units/length.hpp>
```

***

## Building an Arm

{% stepper %}
{% step %}
#### Configure the `SmartMotorControllerConfig`

Use `WithFeedforward(const wpi::math::ArmFeedforward&)` and angular position limits. The motor config must know the gear ratio for accurate position tracking.

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithSubsystem(this)
    .WithMotorInverted(false)
    .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromTeeth({14, 72})))
    .WithFeedback(0.5, 0.0, 0.01)
    .WithFeedforward(wpi::math::ArmFeedforward{           // kS, kG, kV, kA
        0.1_V, 0.0_V, wpi::units::unit_t<wpi::math::ArmFeedforward::kv_unit>{0.5},
        wpi::units::unit_t<wpi::math::ArmFeedforward::ka_unit>{0.01}})
    .WithMechanismLimits(-90_deg, 90_deg)
    .WithStatorCurrentLimit(40_A)
    .WithMOI(0.6_m, 2.0_kg)                     // arm length, arm mass; required for simulation
    .WithStartingPosition(0_deg)                // required for simulation
    .WithTelemetry("ShoulderMotor");
```
{% endstep %}

{% step %}
#### Construct the motor controller

The wrapper takes pointers to the vendor hardware object and to the config. Both must outlive the wrapper, so keep them as subsystem members.

```cpp
rev::spark::SparkMax spark{wpi::CANPort::CAN_S0, 1, rev::spark::SparkMax::MotorType::kBrushless};

yams::motorcontrollers::local::SparkWrapper motor{
    &spark,
    wpi::math::DCMotor::NEO(1),
    &motorConfig
};
```

The wrapper applies the config when it is constructed, so finish configuring `motorConfig` first.
{% endstep %}

{% step %}
#### Create an `ArmConfig`

`WithArmLength`, `WithMinAngle`, and `WithMaxAngle` are required for simulation. All other fields are optional.

```cpp
yams::mechanisms::config::ArmConfig armConfig;
armConfig
    .WithArmLength(0.6_m)
    .WithMinAngle(-100_deg)
    .WithMaxAngle(100_deg)
    .WithTelemetryName("Shoulder");
```
{% endstep %}

{% step %}
#### Construct the `Arm`

```cpp
yams::mechanisms::positional::Arm arm{&armConfig, &motor};
```
{% endstep %}

{% step %}
#### Integrate into a subsystem

The wrapper and the mechanism are held in `std::optional` members and built with `emplace` at the end of the constructor, after the configs are filled in.

```cpp
class ShoulderSubsystem : public wpi::cmd::SubsystemBase {
public:
    ShoulderSubsystem() {
        motorConfig_
            .WithSubsystem(this)
            .WithMotorInverted(false)
            .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
            .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromTeeth({14, 72})))
            .WithFeedback(0.5, 0.0, 0.01)
            .WithFeedforward(wpi::math::ArmFeedforward{
                0.1_V, 0.0_V, wpi::units::unit_t<wpi::math::ArmFeedforward::kv_unit>{0.5},
                wpi::units::unit_t<wpi::math::ArmFeedforward::ka_unit>{0.01}})
            .WithMechanismLimits(-90_deg, 90_deg)
            .WithStatorCurrentLimit(40_A)
            .WithMOI(0.6_m, 2.0_kg)
            .WithStartingPosition(0_deg)
            .WithTelemetry("ShoulderMotor");

        motor_.emplace(&spark_, wpi::math::DCMotor::NEO(1), &motorConfig_);

        armConfig_
            .WithArmLength(0.6_m)
            .WithMinAngle(-100_deg)
            .WithMaxAngle(100_deg)
            .WithTelemetryName("Shoulder");

        arm_.emplace(&armConfig_, &motor_.value());
    }

    wpi::cmd::CommandPtr SetAngleCommand(wpi::units::degree_t angle) {
        return arm_->Run(angle);
    }

    wpi::cmd::CommandPtr RunToAngle(wpi::units::degree_t angle, wpi::units::degree_t tolerance = 2_deg) {
        return arm_->RunTo(angle, tolerance);
    }

    wpi::cmd::Trigger IsNear(wpi::units::degree_t angle, wpi::units::degree_t tolerance = 2_deg) {
        return arm_->IsNear(angle, tolerance);
    }

    wpi::units::degree_t GetAngle() const { return arm_->GetAngle(); }

    void Periodic() override { arm_->UpdateTelemetry(); }
    void SimulationPeriodic() override { arm_->SimIterate(); }

private:
    rev::spark::SparkMax spark_{wpi::CANPort::CAN_S0, 1,
                                rev::spark::SparkMax::MotorType::kBrushless};
    yams::motorcontrollers::SmartMotorControllerConfig motorConfig_;
    std::optional<yams::motorcontrollers::local::SparkWrapper> motor_;
    yams::mechanisms::config::ArmConfig armConfig_;
    std::optional<yams::mechanisms::positional::Arm> arm_;
};
```
{% endstep %}
{% endstepper %}

### Command factories and triggers

| Method                                      | Returns      | Description                                 |
| ------------------------------------------- | ------------ | ------------------------------------------- |
| `Run(degree_t angle)`                       | `CommandPtr` | Move to angle and hold indefinitely         |
| `RunTo(degree_t angle, degree_t tolerance)` | `CommandPtr` | Move to angle, finish when within tolerance |
| `IsNear(degree_t angle, degree_t within)`   | `Trigger`    | True while within tolerance of angle        |
| `Gte(degree_t angle)`                       | `Trigger`    | True when current angle >= threshold        |
| `Lte(degree_t angle)`                       | `Trigger`    | True when current angle <= threshold        |
| `Between(degree_t start, degree_t end)`     | `Trigger`    | True while between the two angles           |
| `Max()`                                     | `Trigger`    | True when at or past upper soft limit       |
| `Min()`                                     | `Trigger`    | True when at or past lower soft limit       |

{% hint style="info" %}
The Java `Trigger` factory was renamed from `isNear(...)` to `near(...)`. In C++ the `Trigger` factory is still `IsNear(...)`.
{% endhint %}

***

## Building an Elevator

{% stepper %}
{% step %}
#### Configure the motor for linear travel

Drum circumference converts rotations to linear distance. Set it on `SmartMotorControllerConfig`, before the linear limits and starting position that depend on it.

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithSubsystem(this)
    .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(9.0))
    .WithMechanismCircumference(wpi::units::meter_t{2 * std::numbers::pi * 0.025})  // 2 * pi * drum radius
    .WithFeedback(1.2, 0.0, 0.05)
    .WithFeedforward(wpi::math::ElevatorFeedforward{      // kS, kG, kV, kA
        0.1_V, 0.0_V, wpi::units::unit_t<wpi::math::ElevatorFeedforward::kv_unit>{0.4},
        wpi::units::unit_t<wpi::math::ElevatorFeedforward::ka_unit>{0.02}})
    .WithMeasurementLimits(0.0_m, 1.2_m)
    .WithStartingPosition(0.0_m)                // required for simulation
    .WithStatorCurrentLimit(60_A)
    .WithTelemetry("ElevatorMotor");
```

> **Note:** Drum radius does NOT go on `ElevatorConfig`; it's set on `SmartMotorControllerConfig` via `WithMechanismCircumference`, `WithMechanismDiameter`, or `WithMechanismRadius`.
{% endstep %}

{% step %}
#### Create an `ElevatorConfig`

`WithCarriageMass`, `WithMinimumHeight`, and `WithMaximumHeight` are required for simulation.

```cpp
yams::mechanisms::config::ElevatorConfig elevatorConfig;
elevatorConfig
    .WithCarriageMass(4.5_kg)           // required for simulation
    .WithMinimumHeight(0.0_m)
    .WithMaximumHeight(1.2_m)
    .WithTelemetryName("Elevator");
```
{% endstep %}

{% step %}
#### Construct the `Elevator`

```cpp
yams::mechanisms::positional::Elevator elevator{&elevatorConfig, &motor};
```
{% endstep %}

{% step %}
#### Command factories and triggers

| Method                                     | Returns      | Description                                  |
| ------------------------------------------ | ------------ | -------------------------------------------- |
| `Run(meter_t height)`                      | `CommandPtr` | Move to height and hold indefinitely         |
| `RunTo(meter_t height, meter_t tolerance)` | `CommandPtr` | Move to height, finish when within tolerance |
| `IsNear(meter_t height, meter_t within)`   | `Trigger`    | True while within tolerance of height        |
| `Gte(meter_t height)`                      | `Trigger`    | True when current height >= threshold        |
| `Lte(meter_t height)`                      | `Trigger`    | True when current height <= threshold        |
| `Between(meter_t start, meter_t end)`      | `Trigger`    | True while between the two heights           |

```cpp
// In periodic:
void Periodic() override { elevator_->UpdateTelemetry(); }
void SimulationPeriodic() override { elevator_->SimIterate(); }
```
{% endstep %}
{% endstepper %}

***

## Building a Pivot

A `Pivot` is a single-jointed rotational mechanism (similar to `Arm`) without a load-bearing arm length. Use it for wrist joints, turrets, or any rotation that doesn't require an arm-length-based MOI estimate.

{% stepper %}
{% step %}
#### Configure the motor

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithSubsystem(this)
    .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(60.0))
    .WithFeedback(4.0, 0.0, 0.1)
    .WithFeedforward(wpi::math::SimpleMotorFeedforward<wpi::units::turns>{
        0.05_V, wpi::units::unit_t<wpi::math::SimpleMotorFeedforward<wpi::units::turns>::kv_unit>{0.1}})
    .WithMechanismLimits(0_deg, 180_deg)
    .WithMOI(0.2_m, 1.0_kg)                     // required for simulation
    .WithStartingPosition(0_deg)                // required for simulation
    .WithStatorCurrentLimit(30_A)
    .WithTelemetry("WristMotor");
```
{% endstep %}

{% step %}
#### Create a `PivotConfig`

`WithMinAngle` and `WithMaxAngle` are required for simulation.

```cpp
yams::mechanisms::config::PivotConfig pivotConfig;
pivotConfig
    .WithMinAngle(0_deg)
    .WithMaxAngle(180_deg)
    .WithTelemetryName("Wrist");
```
{% endstep %}

{% step %}
#### Construct the `Pivot`

```cpp
yams::mechanisms::positional::Pivot pivot{&pivotConfig, &motor};
```
{% endstep %}

{% step %}
#### Command factories and triggers

Same interface as `Arm`: `Run()`, `RunTo()`, `IsNear()`, `Gte()`, `Lte()`, `Between()`, `Max()`, `Min()`.

```cpp
void Periodic() override { pivot_->UpdateTelemetry(); }
void SimulationPeriodic() override { pivot_->SimIterate(); }
```
{% endstep %}
{% endstepper %}

***

## Notes

{% hint style="info" %}
`Run(angle)` holds the setpoint indefinitely. Use it for default positions or when the mechanism should stay put until another command interrupts. `RunTo(angle, tolerance)` ends once the mechanism is within tolerance; use it in command sequences to gate the next step.
{% endhint %}

{% hint style="warning" %}
`WithSubsystem(this)` must be set on `SmartMotorControllerConfig` before constructing the motor controller. Without it, command scheduling will not work correctly.
{% endhint %}

{% hint style="warning" %}
When running in simulation, the mechanism constructors throw a configuration exception if a value the physics model needs is missing: hard limits, starting position, and MOI for `Arm` and `Pivot` (plus arm length for `Arm`); carriage mass, height limits, starting position, and mechanism circumference for `Elevator`.
{% endhint %}

***

## Examples

Complete C++ implementations from the `cpptest` reference project:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ArmSubsystem.cpp" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ElevatorSubsystem.cpp" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/TurretSubsystem.cpp" %}

***

## Related pages

* [Arm (C++)](../c++-reference/mechanisms/arm.md)
* [Elevator (C++)](../c++-reference/mechanisms/elevator.md)
* [Pivot (C++)](../c++-reference/mechanisms/pivot.md)
* [ArmConfig (C++)](../c++-reference/config/arm-config.md)
* [ElevatorConfig (C++)](../c++-reference/config/elevator-config.md)
* [SmartMotorControllerConfig (C++)](../c++-reference/motor-controllers/smart-motor-controller-config.md)
* [Java equivalent: Building an Arm](building-an-arm.md)
* [Java equivalent: Building an Elevator](building-an-elevator.md)
