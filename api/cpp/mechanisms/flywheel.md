# FlyWheel

Namespace: `yams::mechanisms::velocity`\
Header: `<yams/mechanisms/velocity/FlyWheel.h>`\
Inherits: `SmartVelocityMechanism`

**Java equivalent:** [Java FlyWheel](../../java/mechanisms/flywheel.md)

Controls a flywheel or other continuously spinning mechanism. Setpoint is angular velocity or wheel surface speed. Position tracking is not used.

---

## Constructor

```cpp
FlyWheel(config::FlyWheelConfig* config, motorcontrollers::SmartMotorController* smc)
```

Both parameters are non-owning pointers. The `FlyWheelConfig` and `SmartMotorController` must outlive the `FlyWheel` instance.

---

## State Queries

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetVelocity` | `units::degrees_per_second_t GetVelocity() const` | Current angular velocity |

---

## Command Factories

| Method | Signature | Description |
|--------|-----------|-------------|
| `Run` (angular) | `frc2::CommandPtr Run(units::degrees_per_second_t velocity)` | Runs indefinitely at `velocity` |
| `Run` (angular supplier) | `frc2::CommandPtr Run(std::function<units::degrees_per_second_t()> velocity)` | Supplier variant, re-evaluated each iteration |
| `Run` (linear) | `frc2::CommandPtr Run(units::meters_per_second_t surfaceSpeed)` | Controls by wheel surface speed |
| `Run` (linear supplier) | `frc2::CommandPtr Run(std::function<units::meters_per_second_t()> surfaceSpeed)` | Supplier variant |
| `RunTo` (angular) | `frc2::CommandPtr RunTo(units::degrees_per_second_t velocity, units::degrees_per_second_t tolerance = 5.0_deg_per_s)` | Runs until within `tolerance` of `velocity`. Default tolerance: 5°/s. |
| `RunTo` (angular supplier) | `frc2::CommandPtr RunTo(std::function<units::degrees_per_second_t()> velocity, units::degrees_per_second_t tolerance = 5.0_deg_per_s)` | Supplier variant |
| `RunTo` (linear) | `frc2::CommandPtr RunTo(units::meters_per_second_t velocity, units::meters_per_second_t tolerance)` | Surface speed variant; `tolerance` has no default and must be supplied |

{% hint style="info" %}
The C++ `FlyWheel` accepts both angular (`degrees_per_second_t`) and linear surface speed (`meters_per_second_t`) overloads. The linear overload requires `FlyWheelConfig::WithDiameter()` to compute the radius for conversion.
{% endhint %}

---

## Triggers

| Method | Signature | Description |
|--------|-----------|-------------|
| `IsNear` | `frc2::Trigger IsNear(units::degrees_per_second_t velocity, units::degrees_per_second_t within = 5.0_deg_per_s) const` | True when current speed is within `within` of `velocity` |
| `Gte` | `frc2::Trigger Gte(units::degrees_per_second_t velocity)` | True when speed ≥ threshold |
| `Lte` | `frc2::Trigger Lte(units::degrees_per_second_t velocity)` | True when speed ≤ threshold |
| `Between` | `frc2::Trigger Between(units::degrees_per_second_t start, units::degrees_per_second_t end)` | True when speed is in `[start, end]` |

---

## Other Public Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetVelocity` | `void SetVelocity(units::degrees_per_second_t velocity)` | Direct setpoint without command wrapper |
| `SetSurfaceSpeed` | `void SetSurfaceSpeed(units::meters_per_second_t speed)` | Surface speed setpoint; requires `WithDiameter()` in config |
| `GetRelativeMechanismPosition` | `frc::Translation3d GetRelativeMechanismPosition() const` | 3D position of the mechanism |
| `GetConfig` | `const config::FlyWheelConfig& GetConfig() const` | Returns the configuration this flywheel was constructed with |
| `GetMotor` | `motorcontrollers::SmartMotorController* GetMotor()` | Returns the underlying motor controller pointer |
| `SimIterate` | `void SimIterate() override` | Advances flywheel physics simulation one loop iteration |
| `UpdateTelemetry` | `void UpdateTelemetry() override` | Publishes current state to NetworkTables |
| `GetName` | `std::string GetName() const override` | Returns the mechanism name |

---

## Example

```cpp
config::FlyWheelConfig flywheelConfig;
flywheelConfig.WithDiameter(0.1_m)        // required for surface speed overloads
              .WithTelemetryName("Shooter");

motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig.WithMotorInverted(false)
           .WithMotorGearing(gearing::MechanismGearing{1.0})
           .WithFeedback(0.0, 0.0002, 0.0);

// motor is a SmartMotorController* created elsewhere
velocity::FlyWheel flywheel{&flywheelConfig, motor};

// Run at a fixed angular velocity:
frc2::CommandPtr spinUp = flywheel.RunTo(3600_deg_per_s, 50_deg_per_s);

// Or by surface speed (requires WithDiameter in config):
frc2::CommandPtr spinUpLinear = flywheel.RunTo(15_mps, 0.3_mps);
```

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/ShooterSubsystem.cpp#L43-L50" %}

---

## See Also

- [Java FlyWheel](../../java/mechanisms/flywheel.md)
- [C++ FlyWheelConfig](../config/flywheel-config.md)
