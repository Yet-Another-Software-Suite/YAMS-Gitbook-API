# Arm

Namespace: `yams::mechanisms::positional`\
Header: `<yams/mechanisms/positional/Arm.h>`\
Inherits: `SmartPositionalMechanism`

**Java equivalent:** [Java Arm](../../java/mechanisms/arm.md)

Controls a single-jointed arm rotating in the XZ plane. For rotation in the XY plane (turret, shooter hood), use [Pivot](pivot.md).

---

## Constructor

```cpp
Arm(config::ArmConfig* config, motorcontrollers::SmartMotorController* smc)
```

Both parameters are non-owning pointers. The `ArmConfig` and `SmartMotorController` must outlive the `Arm` instance.

---

## State Queries

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetAngle` | `units::degree_t GetAngle() const` | Current mechanism angle from the encoder |

---

## Command Factories

| Method | Signature | Description |
|--------|-----------|-------------|
| `Run` (target) | `frc2::CommandPtr Run(units::degree_t angle)` | Drives to `angle`, runs indefinitely |
| `Run` (supplier) | `frc2::CommandPtr Run(std::function<units::degree_t()> angle)` | Drives to a supplier-provided angle, re-evaluated each iteration |
| `RunTo` (target) | `frc2::CommandPtr RunTo(units::degree_t angle, units::degree_t tolerance = 1.0_deg)` | Drives to `angle`, ends when within `tolerance`. Default tolerance: 1°. |
| `RunTo` (supplier) | `frc2::CommandPtr RunTo(std::function<units::degree_t()> angle, units::degree_t tolerance = 1.0_deg)` | `RunTo` with a supplier target |
| `Set` (duty cycle) | `frc2::CommandPtr Set(double dutycycle)` | Open-loop duty cycle |
| `Set` (supplier) | `frc2::CommandPtr Set(std::function<double()> dutycycle)` | Open-loop duty cycle with supplier |
| `SetVoltage` | `frc2::CommandPtr SetVoltage(units::volt_t volts)` | Open-loop voltage |
| `SetVoltage` (supplier) | `frc2::CommandPtr SetVoltage(std::function<units::volt_t()> volts)` | Open-loop voltage with supplier |

---

## Triggers

| Method | Signature | Description |
|--------|-----------|-------------|
| `IsNear` | `frc2::Trigger IsNear(units::degree_t angle, units::degree_t within = 1.0_deg)` | True when current angle is within `within` of `angle` |
| `Gte` | `frc2::Trigger Gte(units::degree_t angle)` | True when current angle ≥ `angle` |
| `Lte` | `frc2::Trigger Lte(units::degree_t angle)` | True when current angle ≤ `angle` |
| `Between` | `frc2::Trigger Between(units::degree_t start, units::degree_t end)` | True when angle is in `[start, end]` |
| `Max` | `frc2::Trigger Max() override` | True when arm is at its upper hard limit |
| `Min` | `frc2::Trigger Min() override` | True when arm is at its lower hard limit |

---

## Other Public Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetAngle` | `void SetAngle(units::degree_t angle)` | Directly commands the setpoint without a command wrapper |
| `GetRelativeMechanismPosition` | `frc::Translation3d GetRelativeMechanismPosition() const` | 3D end-effector position relative to the mechanism root |
| `GetConfig` | `const config::ArmConfig& GetConfig() const` | Returns the configuration this arm was constructed with |
| `GetMotor` | `motorcontrollers::SmartMotorController* GetMotor()` | Returns the underlying motor controller pointer |
| `GetMechanismLigament` | `frc::MechanismLigament2d* GetMechanismLigament()` | The Mechanism2d visualization ligament |
| `GetMechanismRoot` | `frc::MechanismRoot2d* GetMechanismRoot()` | The Mechanism2d visualization root |
| `SimIterate` | `void SimIterate() override` | Advances arm physics simulation one loop iteration |
| `UpdateTelemetry` | `void UpdateTelemetry() override` | Publishes current state to NetworkTables |
| `VisualizationUpdate` | `void VisualizationUpdate() override` | Syncs Mechanism2d with current encoder reading |
| `GetName` | `std::string GetName() const override` | Returns the mechanism name |

---

## Example

```cpp
config::ArmConfig armConfig;
armConfig.WithMinAngle(-90_deg)
         .WithMaxAngle(90_deg)
         .WithArmLength(0.6_m)
         .WithTelemetryName("Arm");

motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig.WithMotorInverted(false)
           .WithMotorGearing(gearing::MechanismGearing{14.0 / 72.0})
           .WithFeedback(0.5, 0.0, 0.0);

// motor is a SmartMotorController* created elsewhere
positional::Arm arm{&armConfig, motor};

// In the subsystem command factory:
frc2::CommandPtr setAngleCmd = arm.RunTo(45_deg);
```

{% embed url="https://github.com/Yet-Another-Software-Suite/YAMS/tree/master/examples/cpptest" %}
C++ example — includes ArmSubsystem
{% endembed %}

---

## See Also

- [Java Arm](../../java/mechanisms/arm.md)
- [C++ ArmConfig](../config/arm-config.md)
- [Pivot](pivot.md) — rotation in the XY plane
