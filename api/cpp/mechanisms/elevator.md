# Elevator

Namespace: `yams::mechanisms::positional`\
Header: `<yams/mechanisms/positional/Elevator.h>`\
Inherits: `SmartPositionalMechanism`

**Java equivalent:** [Java Elevator](../../java/mechanisms/elevator.md)

Controls a linear elevator mechanism. Position is measured in meters (carriage height).

---

## Constructor

```cpp
Elevator(config::ElevatorConfig* config, motorcontrollers::SmartMotorController* smc)
```

Both parameters are non-owning pointers. The `ElevatorConfig` and `SmartMotorController` must outlive the `Elevator` instance.

---

## State Queries

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetHeight` | `units::meter_t GetHeight() const` | Current carriage height |

---

## Command Factories

| Method | Signature | Description |
|--------|-----------|-------------|
| `Run` (target) | `frc2::CommandPtr Run(units::meter_t height)` | Drives to `height`, runs indefinitely |
| `Run` (supplier) | `frc2::CommandPtr Run(std::function<units::meter_t()> height)` | Supplier target, re-evaluated each iteration |
| `RunTo` (target) | `frc2::CommandPtr RunTo(units::meter_t height, units::meter_t tolerance = 0.01_m)` | Drives to `height`, ends within `tolerance`. Default: 1 cm. |
| `RunTo` (supplier) | `frc2::CommandPtr RunTo(std::function<units::meter_t()> height, units::meter_t tolerance = 0.01_m)` | `RunTo` with supplier |
| `Set` (duty cycle) | `frc2::CommandPtr Set(double dutycycle)` | Open-loop duty cycle |
| `Set` (supplier) | `frc2::CommandPtr Set(std::function<double()> dutycycle)` | Open-loop duty cycle with supplier |
| `SetVoltage` | `frc2::CommandPtr SetVoltage(units::volt_t volts)` | Open-loop voltage output |
| `SetVoltage` (supplier) | `frc2::CommandPtr SetVoltage(std::function<units::volt_t()> volts)` | Open-loop voltage with supplier |

---

## Triggers

| Method | Signature | Description |
|--------|-----------|-------------|
| `IsNear` | `frc2::Trigger IsNear(units::meter_t height, units::meter_t within = 0.01_m)` | True when current height is within `within` of `height` |
| `Gte` | `frc2::Trigger Gte(units::meter_t height)` | True when height ≥ threshold |
| `Lte` | `frc2::Trigger Lte(units::meter_t height)` | True when height ≤ threshold |
| `Between` | `frc2::Trigger Between(units::meter_t start, units::meter_t end)` | True when height is in `[start, end]` |
| `Max` | `frc2::Trigger Max() override` | True when elevator is at its upper hard limit |
| `Min` | `frc2::Trigger Min() override` | True when elevator is at its lower hard limit |

---

## Other Public Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetHeight` | `void SetHeight(units::meter_t height)` | Directly commands the setpoint without a command wrapper |
| `GetRelativeMechanismPosition` | `frc::Translation3d GetRelativeMechanismPosition() const` | 3D carriage position relative to the mechanism root |
| `GetConfig` | `const config::ElevatorConfig& GetConfig() const` | Returns the configuration this elevator was constructed with |
| `GetMotor` | `motorcontrollers::SmartMotorController* GetMotor()` | Returns the underlying motor controller pointer |
| `GetMechanismLigament` | `frc::MechanismLigament2d* GetMechanismLigament()` | The Mechanism2d visualization ligament |
| `GetMechanismRoot` | `frc::MechanismRoot2d* GetMechanismRoot()` | The Mechanism2d visualization root |
| `SimIterate` | `void SimIterate() override` | Advances elevator physics simulation one loop iteration |
| `UpdateTelemetry` | `void UpdateTelemetry() override` | Publishes current state to NetworkTables |
| `VisualizationUpdate` | `void VisualizationUpdate() override` | Syncs Mechanism2d with current encoder reading |
| `GetName` | `std::string GetName() const override` | Returns the mechanism name |

---

## Example

```cpp
config::ElevatorConfig elevatorConfig;
elevatorConfig.WithMinHeight(0.0_m)
              .WithMaxHeight(1.5_m)
              .WithTelemetryName("Elevator");

motorcontrollers::SmartMotorControllerConfig motorConfig;
motorConfig.WithMotorInverted(false)
           .WithMotorGearing(gearing::MechanismGearing{1.0 / 9.0})
           .WithFeedback(1.2, 0.0, 0.0);

// motor is a SmartMotorController* created elsewhere
positional::Elevator elevator{&elevatorConfig, motor};

frc2::CommandPtr goToTop = elevator.RunTo(1.4_m);
```

---

## See Also

- [Java Elevator](../../java/mechanisms/elevator.md)
- [C++ ElevatorConfig](../config/elevator-config.md)
