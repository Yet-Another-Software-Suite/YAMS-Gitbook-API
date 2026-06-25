# SmartMotorControllerConfig

**Namespace:** `yams::motorcontrollers`

`SmartMotorControllerConfig` configures a [`SmartMotorController`](smart-motor-controller.md). All builder methods return `SmartMotorControllerConfig&` for chaining.

**Java equivalent:** [Java SmartMotorControllerConfig](../../java/motor-controllers/smart-motor-controller-config.md)

---

## Enums

### `ControlMode`

| Value | Description |
|-------|-------------|
| `CLOSED_LOOP` | Closed-loop PID/LQR control |
| `OPEN_LOOP` | Direct duty cycle or voltage output |

### `MotorMode`

| Value | Description |
|-------|-------------|
| `COAST` | Motor freewheels when not commanded |
| `BRAKE` | Motor resists motion when not commanded |

### `TelemetryVerbosity`

| Value | Description |
|-------|-------------|
| `NONE` | No telemetry |
| `LOW` | Position and velocity |
| `MEDIUM` | Adds current and voltage |
| `HIGH` | Full state |

### `ClosedLoopControllerSlot`

| Value | Description |
|-------|-------------|
| `SLOT_0` through `SLOT_3` | PID/FF gain sets; motors support up to 4 simultaneous gain configurations |

---

## PID & Feedforward Methods

| Method | Description |
|--------|-------------|
| `WithFeedback(double kP, double kI, double kD, ClosedLoopControllerSlot slot = SLOT_0)` | Set position PID gains |
| `WithKp(double kP, ClosedLoopControllerSlot slot)` | Set proportional gain |
| `WithKi(double kI, ClosedLoopControllerSlot slot)` | Set integral gain |
| `WithKd(double kD, ClosedLoopControllerSlot slot)` | Set derivative gain |
| `WithArmFeedforward(double kS, double kV, double kA, double kG, ClosedLoopControllerSlot slot)` | Arm feedforward (gravity as a function of angle) |
| `WithElevatorFeedforward(double kS, double kV, double kA, ClosedLoopControllerSlot slot)` | Elevator feedforward (constant gravity load) |
| `WithSimpleFeedforward(double kS, double kV, double kA = 0.0, ClosedLoopControllerSlot slot)` | Simple motor feedforward |
| `WithKs(double kS, ClosedLoopControllerSlot slot)` | Set static feedforward gain |
| `WithKv(double kV, ClosedLoopControllerSlot slot)` | Set velocity feedforward gain |
| `WithKa(double kA, ClosedLoopControllerSlot slot)` | Set acceleration feedforward gain |
| `WithKg(double kG, ClosedLoopControllerSlot slot)` | Set gravity feedforward gain |
| `WithLQR(const math::LQRConfig& lqrConfig, ClosedLoopControllerSlot slot)` | LQR controller in place of PID |

---

## Motion Profile Methods

| Method | Description |
|--------|-------------|
| `WithTrapezoidProfile(units::turns_per_second_t maxVelocity, units::turns_per_second_squared_t maxAcceleration)` | Trapezoidal profile for angular mechanisms |
| `WithLinearTrapezoidProfile(units::meters_per_second_t maxVelocity, units::meters_per_second_squared_t maxAcceleration)` | Trapezoidal profile for linear mechanisms |
| `WithVelocityTrapezoidalProfile(units::turns_per_second_t maxVelocity, units::turns_per_second_squared_t maxAcceleration)` | Trapezoidal profile for velocity control |
| `WithExponentialProfile(double kV, double kA, units::volt_t maxInput)` | Exponential profile from system ID gains |
| `WithExponentialProfile(units::volt_t maxVolts, frc::DCMotor motor, units::kilogram_square_meter_t moi)` | Exponential profile from motor model (flywheel or arm) |
| `WithExponentialProfile(units::volt_t maxVolts, frc::DCMotor motor, units::kilogram_t mass, units::meter_t drumRadius)` | Exponential profile from motor model (elevator) |
| `WithExponentialProfile(units::volt_t maxVolts, units::turns_per_second_t maxVelocity, units::turns_per_second_squared_t maxAcceleration)` | Exponential profile from measured physical limits |

---

## Gearing & Measurement Methods

| Method | Description |
|--------|-------------|
| `WithMotorGearing(const gearing::MechanismGearing& gearing)` | Motor-to-mechanism gear ratio |
| `WithMechanismCircumference(units::meter_t circumference)` | Convert rotations to linear distance |
| `WithMechanismCircumference(units::meter_t gearPitch, int teeth)` | Circumference from gear pitch and tooth count |
| `WithMechanismDiameter(units::meter_t diameter)` | Circumference = π × diameter |
| `WithMechanismRadius(units::meter_t radius)` | Circumference = 2π × radius |

---

## Limits & Safety Methods

| Method | Description |
|--------|-------------|
| `WithMechanismLimits(units::turn_t lower, units::turn_t upper)` | Angular soft limits |
| `WithMeasurementLimits(units::meter_t lower, units::meter_t upper)` | Linear soft limits |
| `WithStatorCurrentLimit(units::ampere_t limit)` | Cap motor output current |
| `WithSupplyCurrentLimit(units::ampere_t limit)` | Cap battery draw current |
| `WithTemperatureCutoff(units::celsius_t temperature)` | Disable motor above this temperature |
| `WithClosedLoopMaxVoltage(units::volt_t maxVoltage)` | Cap closed-loop voltage output |

---

## Control Behavior Methods

| Method | Description |
|--------|-------------|
| `WithIdleMode(MotorMode mode)` | Coast or Brake |
| `WithClosedLoopMode()` | Select closed-loop control |
| `WithOpenLoopMode()` | Select open-loop control |
| `WithClosedLoopControlPeriod(units::second_t period)` | Loop period for software PID |
| `WithOpenLoopRampRate(units::second_t rampRate)` | Output ramp time for open-loop |
| `WithClosedLoopRampRate(units::second_t rampRate)` | Output ramp time for closed-loop |
| `WithMotorInverted(bool inverted)` | Invert motor direction |
| `WithEncoderInverted(bool inverted)` | Invert encoder direction |
| `WithContinuousWrapping(units::turn_t min, units::turn_t max)` | Enable continuous input for wrapping mechanisms |

---

## External Encoder Methods

| Method | Description |
|--------|-------------|
| `WithExternalEncoder(std::any encoder)` | Attach an absolute encoder |
| `WithUseExternalFeedbackEncoder(bool use)` | Use external encoder as primary feedback |
| `WithExternalEncoderInverted(bool inverted)` | Invert external encoder reading |
| `WithExternalEncoderConversionFactor(double factor)` | Scale external encoder reading |
| `WithExternalEncoderZeroOffset(units::turn_t zeroOffset)` | Software zero offset for external encoder |
| `WithExternalEncoderGearing(const gearing::MechanismGearing& gearing)` | Gearing between external encoder and mechanism |
| `WithExternalEncoderDiscontinuityPoint(units::turn_t discontinuityPoint)` | Required for SPARK absolute encoders |

---

## Simulation Methods

| Method | Description |
|--------|-------------|
| `WithSimMotor(frc::DCMotor motor)` | Set the simulation motor model |
| `WithMOI(units::kilogram_square_meter_t moi)` | Moment of inertia for simulation |
| `WithMOI(units::meter_t length, units::kilogram_t mass)` | MOI derived from arm dimensions |
| `WithStartingPosition(units::degree_t startingAngle)` | Seed position for angular mechanisms |
| `WithStartingPosition(units::meter_t startingDistance)` | Seed position for linear mechanisms |
| `WithSimClosedLoopController(double kP, double kI, double kD, ClosedLoopControllerSlot slot)` | Separate PID gains used only in simulation |
| `WithSimFeedforward(const frc::ArmFeedforward& ff, ClosedLoopControllerSlot slot)` | Sim-only arm feedforward |
| `WithSimFeedforward(const frc::ElevatorFeedforward& ff, ClosedLoopControllerSlot slot)` | Sim-only elevator feedforward |
| `WithSimFeedforward(const frc::SimpleMotorFeedforward<units::turns>& ff, ClosedLoopControllerSlot slot)` | Sim-only simple motor feedforward |

---

## Other Methods

| Method | Description |
|--------|-------------|
| `WithTelemetry(const std::string& name, TelemetryVerbosity verbosity)` | Configure NetworkTables telemetry |
| `WithSubsystem(frc2::SubsystemBase* subsystem)` | Associate with a WPILib subsystem |
| `WithFollowers(std::vector<std::pair<std::any, bool>> followers)` | Add vendor follower motors |
| `WithLooselyCoupledFollowers(std::vector<SmartMotorController*> followers)` | Add YAMS-managed follower motors |
| `WithVendorConfig(std::any cfg)` | Pass a vendor-specific config object directly |
| `Clone() const` | Returns a deep copy of this config |

---

## Example

```cpp
motorcontrollers::SmartMotorControllerConfig config;
config.WithMotorInverted(false)
      .WithMotorGearing(gearing::MechanismGearing{
          gearing::GearBox::FromTeeth({14, 72})})
      .WithFeedback(0.5, 0.0, 0.0)
      .WithArmFeedforward(0.1, 0.5, 0.01, 0.0, ClosedLoopControllerSlot::SLOT_0)
      .WithMechanismLimits(-90_deg, 90_deg)
      .WithIdleMode(MotorMode::BRAKE)
      .WithTelemetry("ArmMotor", TelemetryVerbosity::HIGH);
```

---

## See Also

- [SmartMotorController](smart-motor-controller.md)
- [Java SmartMotorControllerConfig](../../java/motor-controllers/smart-motor-controller-config.md)
