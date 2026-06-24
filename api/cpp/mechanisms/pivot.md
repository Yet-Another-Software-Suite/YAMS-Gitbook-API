# Pivot

Namespace: `yams::mechanisms::positional`\
Header: `<yams/mechanisms/positional/Pivot.h>`\
Inherits: `SmartPositionalMechanism`

**Java equivalent:** [Java Pivot](../../java/mechanisms/pivot.md)

Controls rotation in the XY plane (turret, shooter hood). For rotation in the XZ plane, use [Arm](arm.md).

---

## Constructor

```cpp
Pivot(config::PivotConfig* config, motorcontrollers::SmartMotorController* smc)
```

Both parameters are non-owning pointers. The `PivotConfig` and `SmartMotorController` must outlive the `Pivot` instance.

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
| `Run` (supplier) | `frc2::CommandPtr Run(std::function<units::degree_t()> angle)` | Supplier target, re-evaluated each iteration |
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
| `Max` | `frc2::Trigger Max() override` | True when pivot is at its upper hard limit |
| `Min` | `frc2::Trigger Min() override` | True when pivot is at its lower hard limit |

---

## Other Public Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetAngle` | `void SetAngle(units::degree_t angle)` | Directly commands the setpoint without a command wrapper |
| `GetRelativeMechanismPosition` | `frc::Translation3d GetRelativeMechanismPosition() const` | 3D end-effector position relative to the mechanism root |
| `GetConfig` | `const config::PivotConfig& GetConfig() const` | Returns the configuration this pivot was constructed with |
| `GetMotor` | `motorcontrollers::SmartMotorController* GetMotor()` | Returns the underlying motor controller pointer |
| `GetMechanismLigament` | `frc::MechanismLigament2d* GetMechanismLigament()` | The Mechanism2d visualization ligament |
| `GetMechanismRoot` | `frc::MechanismRoot2d* GetMechanismRoot()` | The Mechanism2d visualization root |
| `SimIterate` | `void SimIterate() override` | Advances pivot physics simulation one loop iteration |
| `UpdateTelemetry` | `void UpdateTelemetry() override` | Publishes current state to NetworkTables |
| `VisualizationUpdate` | `void VisualizationUpdate() override` | Syncs Mechanism2d with current encoder reading |
| `GetName` | `std::string GetName() const override` | Returns the mechanism name |

---

## Difference from Arm

`Pivot` and `Arm` have identical APIs. The distinction is the plane of rotation and the underlying physics model used for simulation and feedforward:

- `Arm` — rotates in the XZ plane, models gravity acting against the arm length
- `Pivot` — rotates in the XY plane, no gravity component along the rotation axis

Use `PivotConfig` with `Pivot` and `ArmConfig` with `Arm`.

---

## Example

```cpp
config::PivotConfig pivotConfig;
pivotConfig.WithMinAngle(-180_deg)
           .WithMaxAngle(180_deg)
           .WithTelemetryName("Pivot");

// motor is a SmartMotorController* created elsewhere
positional::Pivot pivot{&pivotConfig, motor};

frc2::CommandPtr aimCmd = pivot.RunTo(30_deg);
```

---

## See Also

- [Java Pivot](../../java/mechanisms/pivot.md)
- [C++ PivotConfig](../config/pivot-config.md)
- [Arm](arm.md) — rotation in the XZ plane
