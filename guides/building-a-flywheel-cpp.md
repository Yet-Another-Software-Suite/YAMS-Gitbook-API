# Building a Flywheel Subsystem (C++)

**Goal:** Create a `FlyWheel` subsystem in C++ with velocity control and physics simulation.

---

## Prerequisites

```cpp
#include <yams/YAMS.hpp>

// Or individually:
#include <yams/mechanisms/velocity/FlyWheel.hpp>
#include <yams/mechanisms/config/FlyWheelConfig.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>
```

---

{% stepper %}

{% step %}

#### Configure the motor for velocity control

Use `WithFeedback` for velocity PID and `WithSimpleFeedforward` for steady-state tracking. Set the idle mode to `COAST` so the wheel decelerates freely.

```cpp
yams::motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::COAST)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(1.5)))
    .WithFeedback(0.0003, 0.0, 0.0)
    .WithSimpleFeedforward(0.1, 0.002, 0.0)    // kS, kV, kA
    .WithMOI(0.0508_m, 0.18_kg)                // roller radius, mass — required for simulation
    .WithStatorCurrentLimit(80_A)
    .WithSubsystem(this)
    .WithTelemetry("ShooterMotor");
```

> **Simulation note:** `WithMOI(radius, mass)` sets the moment of inertia for physics simulation. It belongs on `SmartMotorControllerConfig`, not on `FlyWheelConfig`.

{% endstep %}

{% step %}

#### Construct the motor controller

```cpp
yams::motorcontrollers::remote::TalonFXWrapper motor{
    new ctre::phoenix6::hardware::TalonFX(3),
    frc::DCMotor::KrakenX60(1),
    motorConfig
};
```

{% endstep %}

{% step %}

#### Create a `FlyWheelConfig`

`WithRollerDiameter` enables surface speed calculations. All fields are optional.

```cpp
yams::mechanisms::config::FlyWheelConfig flywheelConfig;
flywheelConfig
    .WithRollerDiameter(0.1016_m)    // 4-inch roller
    .WithTelemetryName("Shooter");
```

{% hint style="info" %}
Moment of inertia is set on `SmartMotorControllerConfig` via `WithMOI()`, not on `FlyWheelConfig`. `FlyWheelConfig` only holds the roller diameter, telemetry name, and sim color.
{% endhint %}

{% endstep %}

{% step %}

#### Construct the `FlyWheel`

```cpp
yams::mechanisms::velocity::FlyWheel flyWheel{&flywheelConfig, &motor};
```

{% endstep %}

{% step %}

#### Integrate into a subsystem

```cpp
class ShooterSubsystem : public frc2::SubsystemBase {
public:
    static constexpr units::degrees_per_second_t kShootSpeed{4500.0 * 6.0};   // RPM → deg/s
    static constexpr units::degrees_per_second_t kSpeedTolerance{600.0};       // ~100 RPM

    ShooterSubsystem() {
        motorConfig_
            .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::COAST)
            .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(1.5)))
            .WithFeedback(0.0003, 0.0, 0.0)
            .WithSimpleFeedforward(0.1, 0.002, 0.0)
            .WithMOI(0.0508_m, 0.18_kg)
            .WithStatorCurrentLimit(80_A)
            .WithSubsystem(this)
            .WithTelemetry("ShooterMotor");

        flywheelConfig_
            .WithRollerDiameter(0.1016_m)
            .WithTelemetryName("Shooter");
    }

    /** Spin to a target velocity and hold indefinitely. */
    frc2::CommandPtr SpinUpCommand(units::degrees_per_second_t velocity) {
        return flyWheel_.Run(velocity);
    }

    /** Spin up and finish once within tolerance — use to gate shooting. */
    frc2::CommandPtr SpinUpAndWait(units::degrees_per_second_t velocity) {
        return flyWheel_.RunTo(velocity, kSpeedTolerance);
    }

    /** Trigger that is true whenever the wheel is near the current shoot speed. */
    frc2::Trigger ReadyToShoot() {
        return flyWheel_.IsNear(kShootSpeed, kSpeedTolerance);
    }

    units::degrees_per_second_t GetVelocity() const {
        return flyWheel_.GetVelocity();
    }

    void Periodic() override { flyWheel_.UpdateTelemetry(); }
    void SimulationPeriodic() override { flyWheel_.SimIterate(); }

private:
    yams::motorcontrollers::SmartMotorControllerConfig motorConfig_;
    yams::motorcontrollers::remote::TalonFXWrapper motor_{
        new ctre::phoenix6::hardware::TalonFX(3),
        frc::DCMotor::KrakenX60(1),
        motorConfig_
    };
    yams::mechanisms::config::FlyWheelConfig flywheelConfig_;
    yams::mechanisms::velocity::FlyWheel flyWheel_{&flywheelConfig_, &motor_};
};
```

{% endstep %}

{% step %}

#### Command Factories & Triggers

| Method | Returns | Description |
|--------|---------|-------------|
| `Run(degrees_per_second_t velocity)` | `CommandPtr` | Spin at velocity, hold indefinitely |
| `Run(meters_per_second_t surfaceSpeed)` | `CommandPtr` | Spin at surface speed (requires `WithRollerDiameter`) |
| `RunTo(degrees_per_second_t velocity, tolerance)` | `CommandPtr` | Spin up, finish when within tolerance |
| `IsNear(degrees_per_second_t velocity, within)` | `Trigger` | True while within tolerance of velocity |
| `Gte(degrees_per_second_t velocity)` | `Trigger` | True when speed ≥ threshold |
| `Lte(degrees_per_second_t velocity)` | `Trigger` | True when speed ≤ threshold |

{% endstep %}

{% step %}

#### Sequencing shooter commands

Use `RunTo` to block until the wheel is at speed before feeding:

```cpp
frc2::CommandPtr shoot = shooter_->SpinUpAndWait(ShooterSubsystem::kShootSpeed)
    .AndThen(indexer_->FeedCommand());
```

Or bind the `ReadyToShoot()` trigger to allow feeding whenever the wheel is up:

```cpp
shooter_->ReadyToShoot().WhileTrue(indexer_->FeedCommand());
```

{% endstep %}

{% endstepper %}

---

## Notes

{% hint style="info" %}
`RunTo(velocity, tolerance)` ends once the wheel reaches the target. Use it in command sequences where the next step should not begin until the wheel is at speed.

`IsNear(velocity, tolerance)` (as a `Trigger`) stays true while the wheel remains within tolerance — use it for continuously gated logic such as a conveyor that feeds whenever the shooter is ready.
{% endhint %}

---

## Examples

Complete C++ implementations from the `cpptest` reference project:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ShooterSubsystem.cpp" %}

---

## Related Pages

- [FlyWheel (C++)](../api/cpp/mechanisms/flywheel.md)
- [FlyWheelConfig (C++)](../api/cpp/config/flywheel-config.md)
- [SmartMotorControllerConfig (C++)](../api/cpp/motor-controllers/smart-motor-controller-config.md)
- [TalonFXWrapper (C++)](../api/cpp/motor-controllers/talonfx-wrapper.md)
- [Java equivalent: Building a Flywheel](building-a-flywheel.md)
