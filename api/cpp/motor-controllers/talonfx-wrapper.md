# TalonFXWrapper / TalonFXSWrapper

**Namespace:** `yams::motorcontrollers::remote`\
**Extends:** `SmartMotorController`\
**Header:** `#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>`

`SmartMotorController` implementations for CTRE TalonFX and TalonFXS motor controllers using Phoenix 6. `TalonFXWrapper` targets the TalonFX (Kraken X60, Falcon 500). `TalonFXSWrapper` targets the TalonFXS, which supports third-party brushed and brushless motors such as REV NEO and Minion.

**Java equivalent:** [Java TalonFXWrapper / TalonFXSWrapper](../../java/motor-controllers/talonfx-wrapper.md)

---

## TalonFXWrapper

### Constructor

```cpp
TalonFXWrapper(ctre::phoenix6::hardware::TalonFX* talon,
               frc::DCMotor dcMotor,
               SmartMotorControllerConfig* config)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `talon` | `ctre::phoenix6::hardware::TalonFX*` | TalonFX hardware object. Must outlive this wrapper. |
| `dcMotor` | `frc::DCMotor` | DC motor model used for simulation (e.g. `frc::DCMotor::KrakenX60(1)`). |
| `config` | `SmartMotorControllerConfig*` | Initial configuration to apply. Must outlive this wrapper. |

### Key Methods

The `TalonFXWrapper` implements the full [`SmartMotorController`](smart-motor-controller.md) interface. The table below lists the methods most commonly used directly.

#### Open-Loop Outputs

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetDutyCycle` | `void SetDutyCycle(double dutyCycle)` | Command a duty cycle output in [-1, 1]. |
| `GetDutyCycle` | `double GetDutyCycle()` | Returns the last commanded duty cycle. |
| `SetVoltage` | `void SetVoltage(units::volt_t voltage)` | Command a voltage output. |
| `GetVoltage` | `units::volt_t GetVoltage()` | Returns the last commanded voltage. |

#### Closed-Loop Setpoints

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetPosition` | `void SetPosition(units::turn_t angle)` | Command an angular position setpoint (closed-loop). |
| `SetPosition` | `void SetPosition(units::meter_t distance)` | Command a linear position setpoint. Converts to turns using the configured mechanism circumference. |
| `SetVelocity` | `void SetVelocity(units::turns_per_second_t velocity)` | Command an angular velocity setpoint (closed-loop). |
| `SetVelocity` | `void SetVelocity(units::meters_per_second_t velocity)` | Command a linear velocity setpoint. Converts to turns per second using the configured mechanism circumference. |

#### Encoder Reads

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetMechanismPosition` | `units::turn_t GetMechanismPosition()` | Mechanism-side angular position (after gearing). |
| `GetMechanismVelocity` | `units::turns_per_second_t GetMechanismVelocity()` | Mechanism-side angular velocity. |
| `GetMechanismAcceleration` | `units::turns_per_second_squared_t GetMechanismAcceleration()` | Mechanism-side angular acceleration. |
| `GetRotorPosition` | `units::turn_t GetRotorPosition()` | Raw rotor position (before gearing). |
| `GetRotorVelocity` | `units::turns_per_second_t GetRotorVelocity()` | Raw rotor velocity. |
| `GetMeasurementPosition` | `units::meter_t GetMeasurementPosition()` | Linear measurement position. |
| `GetMeasurementVelocity` | `units::meters_per_second_t GetMeasurementVelocity()` | Linear measurement velocity. |
| `GetMeasurementAcceleration` | `units::meters_per_second_squared_t GetMeasurementAcceleration()` | Linear measurement acceleration. |
| `GetExternalEncoderPosition` | `std::optional<units::degree_t> GetExternalEncoderPosition()` | External encoder position, if configured. |
| `GetExternalEncoderVelocity` | `std::optional<units::degrees_per_second_t> GetExternalEncoderVelocity()` | External encoder velocity, if configured. |

#### Motor Status

| Method | Signature | Description |
|--------|-----------|-------------|
| `GetSupplyCurrent` | `std::optional<units::ampere_t> GetSupplyCurrent()` | Supply (battery-side) current. |
| `GetStatorCurrent` | `units::ampere_t GetStatorCurrent()` | Stator (motor-side) current. |
| `GetTemperature` | `units::celsius_t GetTemperature()` | Motor temperature. |

#### Live Configuration Setters

These methods apply changes immediately without requiring a full `ApplyConfig` call.

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetIdleMode` | `void SetIdleMode(MotorMode mode)` | Coast or Brake. |
| `SetMotorInverted` | `void SetMotorInverted(bool inverted)` | Invert motor output direction. |
| `SetFeedback` | `void SetFeedback(double kP, double kI, double kD)` | Update PID gains in the active slot. |
| `SetFeedforward` | `void SetFeedforward(double kS, double kV, double kA, double kG)` | Update feedforward gains in the active slot. |
| `SetStatorCurrentLimit` | `void SetStatorCurrentLimit(units::ampere_t limit)` | Cap motor stator current. |
| `SetSupplyCurrentLimit` | `void SetSupplyCurrentLimit(units::ampere_t limit)` | Cap battery supply current. |
| `SetMechanismLimits` | `void SetMechanismLimits(units::turn_t lower, units::turn_t upper)` | Update angular soft limits. |
| `SetMechanismLimitsEnabled` | `void SetMechanismLimitsEnabled(bool enabled)` | Enable or disable angular soft limits. |
| `SetMotionProfileMaxVelocity` | Two overloads: `turns_per_second_t` and `meters_per_second_t` | Cap motion profile velocity. |
| `SetMotionProfileMaxAcceleration` | Two overloads: `turns_per_second_squared_t` and `meters_per_second_squared_t` | Cap motion profile acceleration. |
| `SetClosedLoopSlot` | `void SetClosedLoopSlot(ClosedLoopControllerSlot slot)` | Select active PID/FF gain slot. TalonFX supports SLOT_0 through SLOT_2; SLOT_3 is silently ignored. |

#### Advanced Accessors

```cpp
SmartMotorControllerConfig& GetConfig();

// Returns a raw pointer to the underlying ctre::phoenix6::hardware::TalonFX.
void* GetMotorController();

// Returns a raw pointer to the ctre::phoenix6::configs::TalonFXConfiguration.
void* GetMotorControllerConfig();
```

---

## TalonFXSWrapper

**Header:** `#include <yams/motorcontrollers/remote/TalonFXSWrapper.hpp>`

### Motor Arrangement Enum

The TalonFXS drives third-party motors through an external motor port. You must specify which motor type is connected.

```cpp
enum class MotorArrangement {
    Minion,
    NEO,
    NEO550,
    NEOVortex,
    Brushed_2Wire,
    Brushed_3Wire
};
```

### Constructor

```cpp
TalonFXSWrapper(ctre::phoenix6::hardware::TalonFXS* talon,
                frc::DCMotor dcMotor,
                TalonFXSWrapper::MotorArrangement arrangement,
                SmartMotorControllerConfig* config)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `talon` | `ctre::phoenix6::hardware::TalonFXS*` | TalonFXS hardware object. Must outlive this wrapper. |
| `dcMotor` | `frc::DCMotor` | DC motor model used for simulation. |
| `arrangement` | `TalonFXSWrapper::MotorArrangement` | The motor type connected to the TalonFXS. |
| `config` | `SmartMotorControllerConfig*` | Initial configuration to apply. Must outlive this wrapper. |

### Key Methods

`TalonFXSWrapper` exposes the same [`SmartMotorController`](smart-motor-controller.md) interface as `TalonFXWrapper`. The sections above (Open-Loop Outputs, Closed-Loop Setpoints, Encoder Reads, Motor Status, Live Configuration Setters) apply equally.

> **Note:** `SetEncoderInverted` has no effect on TalonFXS — encoder direction follows motor output direction.

#### Advanced Accessors

```cpp
SmartMotorControllerConfig& GetConfig();

// Returns a raw pointer to the underlying ctre::phoenix6::hardware::TalonFXS.
void* GetMotorController();

// Returns a raw pointer to the ctre::phoenix6::configs::TalonFXSConfiguration.
void* GetMotorControllerConfig();
```

---

## Supported Hardware

| Hardware | Wrapper Class | Recommended `frc::DCMotor` Model |
|----------|---------------|----------------------------------|
| TalonFX (Kraken X60) | `TalonFXWrapper` | `frc::DCMotor::KrakenX60(1)` |
| TalonFX (Falcon 500) | `TalonFXWrapper` | `frc::DCMotor::Falcon500(1)` |
| TalonFXS + NEO | `TalonFXSWrapper` | `frc::DCMotor::NEO(1)` |
| TalonFXS + NEO 550 | `TalonFXSWrapper` | `frc::DCMotor::NEO550(1)` |
| TalonFXS + Vortex | `TalonFXSWrapper` | `frc::DCMotor::NEOVortex(1)` |
| TalonFXS + Minion | `TalonFXSWrapper` | `frc::DCMotor::MiniCIM(1)` |

---

## External Encoder Support

### CANcoder

Pass a `CANcoder` reference via `WithExternalEncoder(cancoder)` in the `SmartMotorControllerConfig`. YAMS configures the TalonFX to fuse the CANcoder as the remote feedback source automatically. A discontinuity point is optional; when set it maps to `AbsoluteSensorDiscontinuityPoint` in the CANcoder configuration.

```cpp
ctre::phoenix6::hardware::CANcoder cancoder{9};

yams::motorcontrollers::SmartMotorControllerConfig config{};
config.WithSubsystem(this)
      .WithExternalEncoder(cancoder)
      .WithUseExternalFeedbackEncoder(true)
      .WithFeedback(4.0, 0.0, 0.0)
      .WithMotorGearing(yams::gearing::MechanismGearing{
          yams::gearing::GearBox::FromReductionStages({3.0, 4.0})})
      .WithClosedLoopMode();

ctre::phoenix6::hardware::TalonFX talon{5};
TalonFXWrapper motor{&talon, frc::DCMotor::KrakenX60(1), &config};
```

### CANdi

A `CANdi` can be passed via `WithExternalEncoder(candi)` for both `TalonFXWrapper` and `TalonFXSWrapper`. Because a CANdi has two PWM inputs (PWM1 and PWM2), YAMS cannot determine which port the encoder is wired to automatically. You must supply a vendor config that sets the correct `FeedbackSensorSource` (TalonFX) or `ExternalFeedbackSensorSource` (TalonFXS).

{% hint style="warning" %}
Without a vendor config that explicitly sets the sensor source, YAMS has no way to know which of the two PWM ports the CANdi encoder is connected to and the feedback loop will not work.
{% endhint %}

---

## Code Examples

### TalonFX with CANcoder

```cpp
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/gearing/GearBox.hpp>
#include <yams/gearing/MechanismGearing.hpp>
#include <ctre/phoenix6/TalonFX.hpp>
#include <ctre/phoenix6/CANcoder.hpp>
#include <frc/DCMotor.h>

using namespace yams::motorcontrollers;
using namespace yams::motorcontrollers::remote;
using namespace yams::gearing;
using Cfg = SmartMotorControllerConfig;

// Declare as subsystem members:
//   ctre::phoenix6::hardware::TalonFX    m_talon{5};
//   ctre::phoenix6::hardware::CANcoder   m_cancoder{9};
//   std::optional<TalonFXWrapper>        m_smc;

SmartMotorControllerConfig cfg;
cfg.WithSubsystem(this)
   .WithFeedback(4.0, 0.0, 0.0)
   .WithTrapezoidProfile(units::turns_per_second_t{0.5},
                         units::turns_per_second_squared_t{0.25})
   .WithMotorGearing(MechanismGearing{GearBox::FromReductionStages({3.0, 4.0})})
   .WithIdleMode(Cfg::MotorMode::BRAKE)
   .WithStatorCurrentLimit(40.0_A)
   .WithMotorInverted(false)
   .WithArmFeedforward(0.0, 0.0, 0.0, 0.0)
   .WithExternalEncoder(m_cancoder)
   .WithUseExternalFeedbackEncoder(true)
   .WithClosedLoopMode()
   .WithTelemetry("ArmMotor", Cfg::TelemetryVerbosity::HIGH);

m_smc.emplace(&m_talon, frc::DCMotor::KrakenX60(1), &cfg);
```

### TalonFXS with CANdi

```cpp
#include <yams/motorcontrollers/remote/TalonFXSWrapper.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/gearing/GearBox.hpp>
#include <yams/gearing/MechanismGearing.hpp>
#include <ctre/phoenix6/TalonFXS.hpp>
#include <ctre/phoenix6/CANdi.hpp>
#include <ctre/phoenix6/configs/Configs.hpp>
#include <frc/DCMotor.h>

using namespace yams::motorcontrollers;
using namespace yams::motorcontrollers::remote;
using namespace yams::gearing;
using Cfg = SmartMotorControllerConfig;

// Declare as subsystem members:
//   ctre::phoenix6::hardware::TalonFXS  m_talonFXS{6};
//   ctre::phoenix6::hardware::CANdi     m_candi{10};
//   std::optional<TalonFXSWrapper>      m_smc;

// Configure the TalonFXS to use the correct CANdi PWM port.
ctre::phoenix6::configs::TalonFXSConfiguration talonSConfig{};
talonSConfig.ExternalFeedback.ExternalFeedbackSensorSource =
    ctre::phoenix6::signals::ExternalFeedbackSensorSourceValue::SyncCANdiPWM1;
talonSConfig.ExternalFeedback.ExternalFeedbackRemoteSensorID = m_candi.GetDeviceID();

SmartMotorControllerConfig cfg;
cfg.WithSubsystem(this)
   .WithFeedback(4.0, 0.0, 0.0)
   .WithTrapezoidProfile(units::turns_per_second_t{0.5},
                         units::turns_per_second_squared_t{0.25})
   .WithMotorGearing(MechanismGearing{GearBox::FromReductionStages({3.0, 4.0})})
   .WithIdleMode(Cfg::MotorMode::BRAKE)
   .WithStatorCurrentLimit(40.0_A)
   .WithMotorInverted(false)
   .WithVendorConfig(talonSConfig)
   .WithExternalEncoder(m_candi)
   .WithUseExternalFeedbackEncoder(true)
   .WithClosedLoopMode()
   .WithTelemetry("HoodMotor", Cfg::TelemetryVerbosity::HIGH);

m_smc.emplace(&m_talonFXS, frc::DCMotor::NEO(1),
              TalonFXSWrapper::MotorArrangement::NEO, &cfg);
```

---

## Notes

**Phoenix 6 required.** YAMS only supports Phoenix 6 for TalonFX and TalonFXS. Phoenix 5 is not supported.

**Motion Magic.** When a trapezoidal or exponential profile is configured via `WithTrapezoidProfile` or `WithExponentialProfile`, YAMS uses CTRE Motion Magic control requests internally (`MotionMagicVoltage`, `MotionMagicExpoVoltage`, or the appropriate FOC variant). When a software PID is active instead, profiling runs on the roboRIO.

**FOC support.** Both wrappers support Field-Oriented Control for Kraken X60 and other FOC-capable motors. FOC mode is selected automatically when the configured `frc::DCMotor` model and the applied profile support it. CTRE requires a Phoenix Pro license for FOC on some hardware.

**Configuration retries.** `TalonFXConfiguration` and `TalonFXSConfiguration` are built from `SmartMotorControllerConfig` and applied via the Phoenix 6 configurator. YAMS retries configuration on failure and reports persistent failures via `frc::Alert`.

**Gain slots.** Both wrappers support three closed-loop gain slots (`SLOT_0` through `SLOT_2`). Attempting to activate `SLOT_3` is silently ignored.

**Simulation.** Both wrappers use `frc::sim::DCMotorSim` with the configured moment of inertia for HIL simulation. Call `SetupSimulation()` in your subsystem's simulation init and `SimIterate()` on every simulated loop iteration.

**Lifetime requirements.** The TalonFX/TalonFXS hardware object and the `SmartMotorControllerConfig` must both outlive the wrapper. The recommended pattern is to declare all three as subsystem members (or use `std::optional` for the wrapper) so that construction order and lifetimes are managed correctly by the subsystem.

---

## See Also

- [SmartMotorController](smart-motor-controller.md)
- [SmartMotorControllerConfig](smart-motor-controller-config.md)
- [Java TalonFXWrapper / TalonFXSWrapper](../../java/motor-controllers/talonfx-wrapper.md)
