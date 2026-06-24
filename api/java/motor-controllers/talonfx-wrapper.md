# TalonFXWrapper / TalonFXSWrapper

**Package:** `yams.motorcontrollers.remote`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementations for CTRE TalonFX and TalonFXS controllers. `TalonFXWrapper` targets the TalonFX (Falcon 500, Kraken X60). `TalonFXSWrapper` targets the TalonFXS.

## Constructors

```java
TalonFXWrapper(TalonFX talonFX, DCMotor motor, SmartMotorControllerConfig config)
TalonFXSWrapper(TalonFXS talonFXS, DCMotor motor, SmartMotorControllerConfig config)
```

## Supported Hardware

| Hardware | Class | `DCMotor` Model |
|----------|-------|----------------|
| TalonFX (Kraken X60) | `TalonFXWrapper` | `DCMotor.getKrakenX60(1)` |
| TalonFX (Falcon 500) | `TalonFXWrapper` | `DCMotor.getFalcon500(1)` |
| TalonFXS | `TalonFXSWrapper` | Appropriate CTRE model |

## Example

```java
SmartMotorControllerConfig config = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withStatorCurrentLimit(Amps.of(60))
    .withSupplyCurrentLimit(Amps.of(80))
    .withClosedLoopController(0.3, 0.0, 0.0);

SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(5),
    DCMotor.getKrakenX60(1),
    config
);
```

## CTRE-Specific Notes

**Phoenix 6 required.** YAMS only supports Phoenix 6 for TalonFX and TalonFXS. Phoenix 5 is not supported.

**CANcoder support.** Pass a `CANcoder` via `withExternalEncoder(CANcoder)`. YAMS configures the TalonFX to fuse the CANcoder as the remote feedback source. A discontinuity point is optional for `CANcoder`; when set it maps to `AbsoluteSensorDiscontinuityPoint` in the CANcoder configuration. CANcoder does not have the same dead-zone constraint as SPARK absolute encoders.

**CANdi support.** A `CANdi` can be passed via `withExternalEncoder(CANdi)` for both `TalonFXWrapper` and `TalonFXSWrapper`. Because a CANdi has two PWM inputs (PWM1 and PWM2), YAMS cannot determine which one your encoder is wired to automatically. You must pass a vendor config that sets the correct `FeedbackSensorSource` (TalonFX) or `ExternalFeedbackSensorSource` (TalonFXS) — otherwise the CANdi will not function as an external encoder.

**TalonFX + CANdi:**

```java
TalonFXConfiguration talonConfig = new TalonFXConfiguration();
// Choose SyncCANdiPWM1 or SyncCANdiPWM2 depending on which port the encoder is wired to.
talonConfig.Feedback.FeedbackSensorSource = FeedbackSensorSourceValue.SyncCANdiPWM1;
talonConfig.Feedback.FeedbackRemoteSensorID = candi.getDeviceID();

SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withVendorConfig(talonConfig)
    .withExternalEncoder(candi)
    .withUseExternalFeedbackEncoder(true)
    .withGearing(new MechanismGearing(GearBox.fromRatio(10.0)));

SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(5),
    DCMotor.getKrakenX60(1),
    config
);
```

**TalonFXS + CANdi:**

```java
TalonFXSConfiguration talonSConfig = new TalonFXSConfiguration();
// Choose SyncCANdiPWM1 or SyncCANdiPWM2 depending on which port the encoder is wired to.
talonSConfig.ExternalFeedback.ExternalFeedbackSensorSource =
    ExternalFeedbackSensorSourceValue.SyncCANdiPWM1;
talonSConfig.ExternalFeedback.ExternalFeedbackRemoteSensorID = candi.getDeviceID();

SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withVendorConfig(talonSConfig)
    .withExternalEncoder(candi)
    .withUseExternalFeedbackEncoder(true)
    .withGearing(new MechanismGearing(GearBox.fromRatio(10.0)));

SmartMotorController motor = new TalonFXSWrapper(
    new TalonFXS(6),
    DCMotor.getKrakenX60(1),
    config
);
```

{% hint style="warning" %}
Without a vendor config that explicitly sets the sensor source, YAMS has no way to know which of the two PWM ports the CANdi encoder is connected to and the feedback loop will not work.
{% endhint %}

**Motion Magic.** When a trapezoidal or exponential profile is configured, YAMS uses CTRE Motion Magic control requests internally (`MotionMagicVoltage`, `MotionMagicExpoVoltage`, or the appropriate current-mode variant). When the software PID is active instead, profiling runs on the roboRIO.

**Control requests.** TalonFX uses Phoenix 6 control request objects. YAMS selects the appropriate request type (duty cycle, voltage, position, velocity, torque-current FOC) based on the configuration. Advanced users can inject a custom control request via `withVendorControlRequest(ControlRequest)` in the config.

**Configuration.** `TalonFXConfiguration` is built from `SmartMotorControllerConfig` and applied via `TalonFXConfigurator`. YAMS retries configuration on failure.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia.

## See Also

- [SmartMotorController](smart-motor-controller.md)
- [SmartMotorControllerConfig](smart-motor-controller-config.md)
- [SparkWrapper](spark-wrapper.md)
- [C++ SmartMotorController](../../cpp/motor-controllers/smart-motor-controller.md)
