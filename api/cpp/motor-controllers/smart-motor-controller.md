# SmartMotorController

**Namespace:** `yams::motorcontrollers`

`SmartMotorController` is the abstract base class for all motor controller implementations. Do not instantiate this class directly. Construct concrete instances using vendor-specific subclass constructors such as `SparkWrapper` or `TalonFXWrapper`.

**Java equivalent:** [Java SmartMotorController](../../java/motor-controllers/smart-motor-controller.md)

---

## Pure Virtual Methods

Subclasses must implement all of the following methods.

### Configuration

```cpp
virtual bool ApplyConfig(const SmartMotorControllerConfig& config) = 0;
```

### Simulation

```cpp
virtual void SetupSimulation() = 0;
virtual void SimIterate() = 0;
```

### Encoder Sync

```cpp
virtual void SeedRelativeEncoder() = 0;
virtual void SynchronizeRelativeEncoder() = 0;
```

### Open-Loop Outputs

```cpp
virtual void SetDutyCycle(double dutyCycle) = 0;
virtual double GetDutyCycle() = 0;
virtual void SetVoltage(units::volt_t voltage) = 0;
virtual units::volt_t GetVoltage() = 0;
```

### Closed-Loop Setpoints

```cpp
virtual void SetPosition(units::turn_t angle) = 0;
virtual void SetPosition(units::meter_t distance) = 0;
virtual void SetVelocity(units::turns_per_second_t velocity) = 0;
virtual void SetVelocity(units::meters_per_second_t velocity) = 0;
```

### Encoder Reads

```cpp
virtual units::turn_t GetMechanismPosition() = 0;
virtual units::turns_per_second_t GetMechanismVelocity() = 0;
virtual units::turns_per_second_squared_t GetMechanismAcceleration() = 0;
virtual units::turn_t GetRotorPosition() = 0;
virtual units::turns_per_second_t GetRotorVelocity() = 0;
virtual units::meter_t GetMeasurementPosition() = 0;
virtual units::meters_per_second_t GetMeasurementVelocity() = 0;
virtual units::meters_per_second_squared_t GetMeasurementAcceleration() = 0;
virtual std::optional<units::degree_t> GetExternalEncoderPosition() = 0;
virtual std::optional<units::degrees_per_second_t> GetExternalEncoderVelocity() = 0;
```

### Motor Status

```cpp
virtual std::optional<units::ampere_t> GetSupplyCurrent() = 0;
virtual units::ampere_t GetStatorCurrent() = 0;
virtual units::celsius_t GetTemperature() = 0;
virtual frc::DCMotor GetDCMotor() = 0;
```

### Live Configuration Setters

| Method | Signature | Description |
|--------|-----------|-------------|
| `SetIdleMode` | `virtual void SetIdleMode(MotorMode mode) = 0` | Coast or Brake |
| `SetMotorInverted` | `virtual void SetMotorInverted(bool inverted) = 0` | Invert motor direction |
| `SetEncoderInverted` | `virtual void SetEncoderInverted(bool inverted) = 0` | Invert encoder direction |
| `SetKp` | `virtual void SetKp(double kP) = 0` | Proportional PID gain |
| `SetKi` | `virtual void SetKi(double kI) = 0` | Integral PID gain |
| `SetKd` | `virtual void SetKd(double kD) = 0` | Derivative PID gain |
| `SetFeedback` | `virtual void SetFeedback(double kP, double kI, double kD) = 0` | Set all PID gains at once |
| `SetKs` | `virtual void SetKs(double kS) = 0` | Static feedforward gain |
| `SetKv` | `virtual void SetKv(double kV) = 0` | Velocity feedforward gain |
| `SetKa` | `virtual void SetKa(double kA) = 0` | Acceleration feedforward gain |
| `SetKg` | `virtual void SetKg(double kG) = 0` | Gravity feedforward gain |
| `SetFeedforward` | `virtual void SetFeedforward(double kS, double kV, double kA, double kG) = 0` | Set all feedforward gains at once |
| `SetStatorCurrentLimit` | `virtual void SetStatorCurrentLimit(units::ampere_t limit) = 0` | Cap motor output current |
| `SetSupplyCurrentLimit` | `virtual void SetSupplyCurrentLimit(units::ampere_t limit) = 0` | Cap battery draw current |
| `SetClosedLoopRampRate` | `virtual void SetClosedLoopRampRate(units::second_t rampRate) = 0` | Ramp time for closed-loop output |
| `SetOpenLoopRampRate` | `virtual void SetOpenLoopRampRate(units::second_t rampRate) = 0` | Ramp time for open-loop output |
| `SetMechanismLimits` | `virtual void SetMechanismLimits(units::turn_t lower, units::turn_t upper) = 0` | Update angular soft limits |
| `SetMechanismLimitsEnabled` | `virtual void SetMechanismLimitsEnabled(bool enabled) = 0` | Enable or disable soft limits |
| `SetMotionProfileMaxVelocity` | Two overloads: `turns_per_second_t` and `meters_per_second_t` | Cap motion profile velocity |
| `SetMotionProfileMaxAcceleration` | Two overloads: `turns_per_second_squared_t` and `meters_per_second_squared_t` | Cap motion profile acceleration |
| `SetClosedLoopSlot` | `virtual void SetClosedLoopSlot(ClosedLoopControllerSlot slot) = 0` | Select active PID/FF gain slot |

---

## Non-Virtual Public Methods

### Simulation

```cpp
void SetSimSupplier(std::shared_ptr<SimSupplier> supplier);
SimSupplier* GetSimSupplier() const;
```

### Closed-Loop Controller Thread

```cpp
void StartClosedLoopController();
void StopClosedLoopController();
void IterateClosedLoopController();
```

### Telemetry

```cpp
void SetupTelemetry(std::shared_ptr<nt::NetworkTable> dataTable,
                    std::shared_ptr<nt::NetworkTable> tuningTable);
void SetupTelemetry();   // Uses default table names
void UpdateTelemetry();
SmartMotorController& WithTelemetry(telemetry::SmartMotorControllerTelemetryConfig config);
```

### Accessors

```cpp
ClosedLoopControllerSlot GetClosedLoopControllerSlot() const;
std::optional<units::turn_t> GetMechanismPositionSetpoint() const;
std::optional<units::turns_per_second_t> GetMechanismSetpointVelocity() const;
std::optional<units::meter_t> GetMeasurementPositionSetpoint() const;
std::optional<units::meters_per_second_t> GetMeasurementSetpointVelocity() const;
std::string GetName() const;
bool IsMotor(const frc::DCMotor& a, const frc::DCMotor& b) const;
void CheckConfigSafety();
void Close();
virtual SmartMotorControllerConfig& GetConfig() = 0;
virtual void* GetMotorController() = 0;
virtual void* GetMotorControllerConfig() = 0;
```

---

## SimSupplier Interface

`SimSupplier` is a pure virtual interface for physics simulation backends. Implement this interface to provide custom physics, or use one of the three built-in implementations:

| Implementation | Description |
|----------------|-------------|
| `simulation::DCMotorSimSupplier` | Generic DC motor physics |
| `simulation::ArmSimSupplier` | Single-jointed arm physics |
| `simulation::ElevatorSimSupplier` | Linear elevator physics |

---

## CTRE-Specific Notes

### CANdi as External Encoder (TalonFXWrapper / TalonFXSWrapper)

A `CANdi` can be passed via `WithExternalEncoder(candi)` for both `TalonFXWrapper` and `TalonFXSWrapper`. Because a CANdi has two PWM inputs (PWM1 and PWM2), YAMS cannot determine which port the encoder is wired to automatically. You must pass a vendor config that sets the correct feedback sensor source — otherwise the CANdi will not function as an external encoder.

**TalonFXWrapper + CANdi:**

```cpp
ctre::phoenix6::configs::TalonFXConfiguration talonConfig{};
// Choose SyncCANdiPWM1 or SyncCANdiPWM2 depending on which port the encoder is wired to.
talonConfig.Feedback.FeedbackSensorSource =
    ctre::phoenix6::signals::FeedbackSensorSourceValue::SyncCANdiPWM1;
talonConfig.Feedback.FeedbackRemoteSensorID = candi.GetDeviceID();

yams::motorcontrollers::SmartMotorControllerConfig config{};
config.WithVendorConfig(talonConfig)
      .WithExternalEncoder(candi)
      .WithUseExternalFeedbackEncoder(true)
      .WithGearing(yams::MechanismGearing{yams::GearBox::FromRatio(10.0)});

TalonFXWrapper motor{&talon, frc::DCMotor::KrakenX60(1), config};
```

**TalonFXSWrapper + CANdi:**

```cpp
ctre::phoenix6::configs::TalonFXSConfiguration talonSConfig{};
// Choose SyncCANdiPWM1 or SyncCANdiPWM2 depending on which port the encoder is wired to.
talonSConfig.ExternalFeedback.ExternalFeedbackSensorSource =
    ctre::phoenix6::signals::ExternalFeedbackSensorSourceValue::SyncCANdiPWM1;
talonSConfig.ExternalFeedback.ExternalFeedbackRemoteSensorID = candi.GetDeviceID();

yams::motorcontrollers::SmartMotorControllerConfig config{};
config.WithVendorConfig(talonSConfig)
      .WithExternalEncoder(candi)
      .WithUseExternalFeedbackEncoder(true)
      .WithGearing(yams::MechanismGearing{yams::GearBox::FromRatio(10.0)});

TalonFXSWrapper motor{&talonfxs, frc::DCMotor::KrakenX60(1), config};
```

{% hint style="warning" %}
Without a vendor config that explicitly sets the sensor source, YAMS has no way to know which of the two PWM ports the CANdi encoder is connected to and the feedback loop will not work.
{% endhint %}

---

## See Also

- [SmartMotorControllerConfig](smart-motor-controller-config.md)
- [Java SmartMotorController](../../java/motor-controllers/smart-motor-controller.md)
