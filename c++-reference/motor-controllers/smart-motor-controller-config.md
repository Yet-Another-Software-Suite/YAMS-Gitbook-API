# SmartMotorControllerConfig

**Namespace:** `yams::motorcontrollers`\
**Header:** `#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>`

`SmartMotorControllerConfig` configures a [`SmartMotorController`](smart-motor-controller.md). All builder methods return `SmartMotorControllerConfig&` for chaining. Units are `wpi::units::*` types and motor models are `wpi::math::DCMotor`. The wrappers take the config by pointer, so it must outlive the wrapper.

**Java equivalent:** [Java SmartMotorControllerConfig](../../java-reference/motor-controllers/smart-motor-controller-config.md)

***

## Enums

### `ControlMode`

| Value         | Description                         |
| ------------- | ----------------------------------- |
| `CLOSED_LOOP` | Closed-loop PID/[LQR](../math/lqr-controller.md) control         |
| `OPEN_LOOP`   | Direct duty cycle or voltage output |

### `MotorMode`

| Value   | Description                             |
| ------- | --------------------------------------- |
| `COAST` | Motor freewheels when not commanded     |
| `BRAKE` | Motor resists motion when not commanded |

### `TelemetryVerbosity`

| Value    | Description                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------- |
| `NONE`   | No telemetry                                                                                      |
| `LOW`    | Setpoints, positions, velocities and accelerations, active slot                                   |
| `MEDIUM` | Adds output voltage, stator and supply current                                                    |
| `HIGH`   | Full state, tunable values and gains. Also registers the live tuning command (see [SmartMotorController](smart-motor-controller.md#telemetry-and-live-tuning)) |

See [SmartMotorControllerTelemetryConfig](smart-motor-controller-telemetry-config.md#verbosity-presets) for the exact fields.

### `ClosedLoopControllerSlot`

| Value                     | Description                                                                                  |
| ------------------------- | -------------------------------------------------------------------------------------------- |
| `SLOT_0` through `SLOT_3` | PID/FF gain sets. SPARKs support all four; TalonFX and TalonFXS support `SLOT_0` to `SLOT_2` |

***

## PID & Feedforward Methods

| Method                                                                                                                       | Description                                      |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `WithFeedback(double kP, double kI, double kD, ClosedLoopControllerSlot slot = SLOT_0)`                                      | Set PID gains                                    |
| `WithFeedforward(const wpi::math::ArmFeedforward& ff, ClosedLoopControllerSlot slot = SLOT_0)`                               | Arm feedforward (gravity as a function of angle) |
| `WithFeedforward(const wpi::math::ElevatorFeedforward& ff, ClosedLoopControllerSlot slot = SLOT_0)`                          | Elevator feedforward (constant gravity load)     |
| `WithFeedforward(const wpi::math::SimpleMotorFeedforward<wpi::units::turns>& ff, ClosedLoopControllerSlot slot = SLOT_0)`    | Simple motor feedforward                         |
| `WithLQR(const math::LQRConfig& lqrConfig, ClosedLoopControllerSlot slot = SLOT_0)`                                          | [LQR controller](../math/lqr-controller.md) in place of PID (runs on the SystemCore) |
| `WithLinearClosedLoopController(bool linear)`                                                                                | Run the closed loop in meters instead of rotations (needs a circumference) |
| `WithClosedLoopTolerance(wpi::units::turn_t tolerance)`                                                                      | Software PID tolerance in mechanism rotations    |
| `WithClosedLoopTolerance(wpi::units::meter_t tolerance)`                                                                     | Software PID tolerance as a distance             |

***

## Motion Profile Methods

| Method                                                                                                                                                       | Description                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| `WithTrapezoidProfile(wpi::units::turns_per_second_t maxVelocity, wpi::units::turns_per_second_squared_t maxAcceleration)`                                   | Trapezoidal profile for angular position control       |
| `WithLinearTrapezoidProfile(wpi::units::meters_per_second_t maxVelocity, wpi::units::meters_per_second_squared_t maxAcceleration)`                           | Trapezoidal profile for linear position control        |
| `WithVelocityTrapezoidProfile(wpi::units::turns_per_second_squared_t maxAcceleration, wpi::units::angular_jerk::turns_per_second_cubed_t maxJerk)`             | Profiles an angular velocity setpoint (acceleration and jerk limits) |
| `WithVelocityTrapezoidProfile(wpi::units::meters_per_second_squared_t maxAcceleration, meters_per_second_cubed_t maxJerk)`                                   | Profiles a linear velocity setpoint; switches to linear closed loop |
| `WithExponentialProfile(double kV, double kA, wpi::units::volt_t maxInput)`                                                                                  | Exponential profile from system ID gains               |
| `WithExponentialProfile(wpi::units::volt_t maxVolts, wpi::math::DCMotor motor, wpi::units::kilogram_square_meter_t moi)`                                     | Exponential profile from motor model (flywheel or arm) |
| `WithExponentialProfile(wpi::units::volt_t maxVolts, wpi::math::DCMotor motor, wpi::units::kilogram_t mass, wpi::units::meter_t drumRadius)`                  | Exponential profile from motor model (elevator); sets the circumference and linear closed loop |
| `WithExponentialProfile(wpi::units::volt_t maxVolts, wpi::units::turns_per_second_t maxVelocity, wpi::units::turns_per_second_squared_t maxAcceleration)`    | Exponential profile from measured physical limits      |
| `WithExponentialProfile(wpi::math::ExponentialProfile<wpi::units::turns, wpi::units::volts>::Constraints constraints)`                                       | Exponential profile from prebuilt constraints          |
| `WithLinearExponentialProfile(double kV, double kA, wpi::units::volt_t maxInput)`                                                                           | Meters based exponential profile (needs a circumference) |

`meters_per_second_cubed_t` is a type alias declared in `yams::motorcontrollers`.

***

## Gearing & Measurement Methods

| Method                                                                 | Description                                         |
| ---------------------------------------------------------------------- | --------------------------------------------------- |
| `WithMotorGearing(const gearing::MechanismGearing& gearing)`           | Motor-to-mechanism gear ratio                       |
| `WithMotorGearing(double reductionRatio)`                              | Gear ratio as a number (3.0 for 3:1)                |
| `WithCascadingElevatorStages(int stages)`                              | Divide the configured gearing by the number of cascading stages |
| `WithMechanismCircumference(wpi::units::meter_t circumference)`        | Convert rotations to linear distance                |
| `WithMechanismCircumference(wpi::units::meter_t gearPitch, int teeth)` | Circumference from gear pitch and tooth count       |
| `WithMechanismDiameter(wpi::units::meter_t diameter)`                  | Circumference = π × diameter                        |
| `WithMechanismRadius(wpi::units::meter_t radius)`                      | Circumference = 2π × radius                         |

***

## Limits & Safety Methods

| Method                                                                         | Description                                          |
| ------------------------------------------------------------------------------ | ---------------------------------------------------- |
| `WithMechanismLimits(wpi::units::turn_t lower, wpi::units::turn_t upper)`      | Angular soft limits                                  |
| `WithMeasurementLimits(wpi::units::meter_t lower, wpi::units::meter_t upper)`  | Linear soft limits (set the circumference first)     |
| `WithStatorCurrentLimit(wpi::units::ampere_t limit)`                           | Cap motor output current                             |
| `WithSupplyCurrentLimit(wpi::units::ampere_t limit)`                           | Cap battery draw current                             |
| `WithTemperatureCutoff(wpi::units::celsius_t temperature)`                     | Disable motor above this temperature                 |
| `WithClosedLoopMaxVoltage(wpi::units::volt_t maxVoltage)`                      | Cap closed-loop voltage output                       |
| `WithVoltageCompensation(wpi::units::volt_t voltage)`                          | Nominal voltage the output is compensated for        |
| `WithFeedbackSynchronizationThreshold(wpi::units::turn_t threshold)`           | Resync the relative encoder to the absolute encoder beyond this error |

***

## Control Behavior Methods

| Method                                                                   | Description                                                          |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| `WithZeroPower(MotorMode mode)`                                          | Coast or Brake                                                       |
| `WithClosedLoopMode()`                                                   | Select closed-loop control                                           |
| `WithOpenLoopMode()`                                                     | Select open-loop control                                             |
| `WithClosedLoopControlPeriod(wpi::units::second_t period)`               | Period of the SystemCore closed-loop thread                          |
| `WithResetPreviousConfig(bool reset)`                                    | Reset the controller's previous configuration first (default true)  |
| `WithOpenLoopRampRate(wpi::units::second_t rampRate)`                    | Output ramp time for open-loop                                       |
| `WithClosedLoopRampRate(wpi::units::second_t rampRate)`                  | Output ramp time for closed-loop                                     |
| `WithMotorInverted(bool inverted)`                                       | Invert motor direction                                               |
| `WithEncoderInverted(bool inverted)`                                     | Invert encoder direction                                             |
| `WithContinuousWrapping(wpi::units::turn_t min, wpi::units::turn_t max)` | Enable continuous input for wrapping mechanisms                      |

***

## External Encoder Methods

| Method                                                                         | Description                                                    |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| `WithExternalEncoder(std::any encoder)`                                        | Attach an external encoder, passed as a pointer (see below)    |
| `WithUseExternalFeedbackEncoder(bool use)`                                     | Use the external encoder as the closed-loop feedback source    |
| `WithExternalEncoderInverted(bool inverted)`                                   | Invert the external encoder reading                            |
| `WithExternalEncoderConversionFactor(double factor)`                           | Override the conversion factor (prefer the gearing methods)    |
| `WithExternalEncoderZeroOffset(wpi::units::turn_t zeroOffset)`                 | Zero offset for the external encoder                           |
| `WithExternalEncoderZeroOffset(wpi::units::meter_t zeroOffset)`                | Zero offset as a distance (needs a circumference)              |
| `WithExternalEncoderGearing(const gearing::MechanismGearing& gearing)`         | Gearing between the external encoder and the mechanism         |
| `WithExternalEncoderGearing(double reductionRatio)`                            | Same, as a number                                              |
| `WithExternalEncoderDiscontinuityPoint(wpi::units::turn_t discontinuityPoint)` | Wrap point, exactly `0.5_tr` (reads \[-0.5, 0.5)) or `1_tr` (reads \[0, 1)) |

The zero offset is read back with `GetExternalEncoderZeroOffset()`.

Accepted external encoder types per wrapper:

| Wrapper                             | `WithExternalEncoder(...)` argument                                                                                                                                  |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TalonFXWrapper`, `TalonFXSWrapper` | `ctre::phoenix6::hardware::CANcoder*`, `ctre::phoenix6::hardware::CANdi*` (CANdi needs a vendor config, see [TalonFXWrapper](talonfx-wrapper.md#candi))              |
| `SparkWrapper`                      | `rev::spark::SparkAbsoluteEncoder*`, `rev::spark::SparkMaxAlternateEncoder*`, `rev::spark::SparkFlexExternalEncoder*`, `rev::detached::DetachedEncoder*`, `rev::detached::SplineEncoder*` |
| `NovaWrapper`                       | `thrifty::Motor::FeedbackSensorType::ABS`, `thrifty::Motor::FeedbackSensorType::QUAD`, `thrifty::CanEncoder*`                                                       |

***

## Simulation Methods

| Method                                                                                                                       | Description                                       |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `WithSimMotor(wpi::math::DCMotor motor)`                                                                                     | Set the simulation motor model                    |
| `WithSimulationPeriod(wpi::units::second_t period)`                                                                          | Rate at which `SimIterate()` steps the physics (default 20 ms) |
| `WithMOI(wpi::units::kilogram_square_meter_t moi)`                                                                           | Moment of inertia for simulation                  |
| `WithMOI(wpi::units::meter_t length, wpi::units::kilogram_t mass)`                                                           | MOI estimated from arm length and mass            |
| `WithStartingPosition(wpi::units::degree_t startingAngle)`                                                                   | Seed position for angular mechanisms              |
| `WithStartingPosition(wpi::units::meter_t startingDistance)`                                                                 | Seed position for linear mechanisms (set the circumference first) |
| `WithSimStartingPosition(wpi::units::degree_t startingAngle)`                                                                | Starting angle used only in simulation            |
| `WithSimStartingPosition(wpi::units::meter_t startingDistance)`                                                              | Starting distance used only in simulation         |
| `WithSimClosedLoopController(double kP, double kI, double kD, ClosedLoopControllerSlot slot = SLOT_0)`                       | PID gains used only in simulation                 |
| `WithSimClosedLoopController(const math::LQRConfig& lqrConfig, ClosedLoopControllerSlot slot = SLOT_0)`                      | [LQR](../math/lqr-controller.md) used only in simulation                       |
| `WithSimFeedforward(const wpi::math::ArmFeedforward& ff, ClosedLoopControllerSlot slot = SLOT_0)`                            | Sim-only arm feedforward                          |
| `WithSimFeedforward(const wpi::math::ElevatorFeedforward& ff, ClosedLoopControllerSlot slot = SLOT_0)`                       | Sim-only elevator feedforward                     |
| `WithSimFeedforward(const wpi::math::SimpleMotorFeedforward<wpi::units::turns>& ff, ClosedLoopControllerSlot slot = SLOT_0)` | Sim-only simple motor feedforward                 |
| `WithSimTrapezoidProfile(wpi::units::turns_per_second_t maxVelocity, wpi::units::turns_per_second_squared_t maxAcceleration)` | Sim-only angular trapezoidal profile             |
| `WithSimTrapezoidProfile(wpi::units::meters_per_second_t maxVelocity, wpi::units::meters_per_second_squared_t maxAcceleration)` | Sim-only linear trapezoidal profile            |
| `WithSimExponentialProfile(wpi::math::ExponentialProfile<wpi::units::turns, wpi::units::volts>::Constraints constraints)`    | Sim-only exponential profile                      |

***

## Telemetry Methods

| Method                                                                                                  | Description                                                     |
| ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `WithTelemetry(const std::string& name, TelemetryVerbosity verbosity = TelemetryVerbosity::HIGH)`       | Configure NetworkTables telemetry                               |
| `WithTelemetry(TelemetryVerbosity verbosity)`                                                           | Same, under the name `motor`                                    |
| `WithTelemetry(const std::string& name, telemetry::SmartMotorControllerTelemetryConfig telemetryConfig)` | Field-level telemetry; sets the verbosity to HIGH so live tuning is available. Give each motor controller its own telemetry config |

Live tuning is registered automatically at `TelemetryVerbosity::HIGH` when a subsystem is set with `WithSubsystem(...)`. There is no separate opt-in call in C++.

***

## Other Methods

| Method                                                                      | Description                                                  |
| --------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `WithSubsystem(wpi::cmd::SubsystemBase* subsystem)`                         | Associate with a Commands v2 subsystem (throws a `SmartMotorControllerConfigurationException` if already set) |
| `WithFollowers(std::vector<std::pair<std::any, bool>> followers)`           | Tightly coupled vendor followers: (hardware pointer, oppose master) pairs |
| `WithLooselyCoupledFollowers(std::vector<SmartMotorController*> followers)` | YAMS-managed followers that mirror setpoints                 |
| `WithVendorConfig(std::any cfg)`                                            | Vendor config applied first; YAMS options override it        |
| `WithVendorControlRequest(std::any req)`                                    | Phoenix 6 position or velocity control request to use (TalonFX/TalonFXS) |
| `ClearFollowers()`                                                          | Remove the tightly coupled followers                         |
| `Clone() const`                                                             | Returns a deep copy of this config                           |

Vendor config types: `ctre::phoenix6::configs::TalonFXConfiguration` (TalonFX), `ctre::phoenix6::configs::TalonFXSConfiguration` (TalonFXS), `rev::spark::SparkMaxConfig` / `rev::spark::SparkFlexConfig` (SPARK), `thrifty::NovaConfigBatch` (Nova).

Follower types: `ctre::phoenix6::hardware::TalonFX*` or `TalonFXS*` (CTRE wrappers), `rev::spark::SparkMax*` or `rev::spark::SparkFlex*` (SparkWrapper), `thrifty::Nova*` (NovaWrapper).

***

## Getters

Every option has a getter, for example `GetKp(slot)`, `GetSlotGains(slot)`, `GetMotorGearing()`, `GetMechanismCircumference()`, `GetMechanismLowerLimit()`, `GetStartingPosition()`, `GetExternalEncoderZeroOffset()`, `GetExternalEncoderDiscontinuityPoint()`, `GetTelemetryName()`, `GetVerbosity()`, `GetSubsystem()` and `GetMOI()`. Unset options return an empty `std::optional`. The unit helpers `ConvertFromMechanism(...)`, `ConvertToMechanism(...)`, `ConvertToVoltage(motor, force)` and `ConvertToCurrent(motor, force)` convert between mechanism rotations, distances and feedforward forces.

***

## Example

```cpp
#include <yams/gearing/GearBox.hpp>
#include <yams/gearing/MechanismGearing.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>

using namespace yams::motorcontrollers;
using namespace yams::gearing;
using Cfg = SmartMotorControllerConfig;

// m_motorConfig is a subsystem member so it outlives the wrapper.
m_motorConfig.WithSubsystem(this)
    .WithMotorInverted(false)
    .WithMotorGearing(MechanismGearing{GearBox::FromTeeth({14, 72})})
    .WithFeedback(0.5, 0.0, 0.0)
    .WithFeedforward(wpi::math::ArmFeedforward{
                         wpi::units::volt_t{0.1}, wpi::units::volt_t{0},
                         wpi::units::unit_t<wpi::math::ArmFeedforward::kv_unit>{0.5},
                         wpi::units::unit_t<wpi::math::ArmFeedforward::ka_unit>{0.01}},
                     Cfg::ClosedLoopControllerSlot::SLOT_0)
    .WithMechanismLimits(wpi::units::degree_t{-90}, wpi::units::degree_t{90})
    .WithZeroPower(Cfg::MotorMode::BRAKE)
    .WithClosedLoopMode()
    .WithTelemetry("ArmMotor", Cfg::TelemetryVerbosity::HIGH);
```

***

## See Also

* [SmartMotorController](smart-motor-controller.md)
* [SmartMotorControllerTelemetryConfig](smart-motor-controller-telemetry-config.md)
* [Java SmartMotorControllerConfig](../../java-reference/motor-controllers/smart-motor-controller-config.md)
