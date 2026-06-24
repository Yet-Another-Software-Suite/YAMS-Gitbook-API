# TalonFXWrapper / TalonFXSWrapper

**Package:** `yams.motorcontrollers.remote`\
**Extends:** `SmartMotorController`

`SmartMotorController` implementations for CTRE TalonFX and TalonFXS controllers. `TalonFXWrapper` targets the TalonFX (Falcon 500, Kraken X60). `TalonFXSWrapper` targets the TalonFXS. Construct via `SmartMotorController.create()` or directly via the constructor.

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

**CANdi support.** A `CANdi` can also be passed as the external encoder object for TalonFXS.

**Motion Magic.** When a trapezoidal or exponential profile is configured, YAMS uses CTRE Motion Magic control requests internally (`MotionMagicVoltage`, `MotionMagicExpoVoltage`, or the appropriate current-mode variant). When the software PID is active instead, profiling runs on the roboRIO.

**Control requests.** TalonFX uses Phoenix 6 control request objects. YAMS selects the appropriate request type (duty cycle, voltage, position, velocity, torque-current FOC) based on the configuration. Advanced users can inject a custom control request via `withVendorControlRequest(ControlRequest)` in the config.

**Configuration.** `TalonFXConfiguration` is built from `SmartMotorControllerConfig` and applied via `TalonFXConfigurator`. YAMS retries configuration on failure.

**Simulation.** Uses `DCMotorSim` with the configured moment of inertia.

## See Also

- [SmartMotorController](smart-motor-controller.md)
- [SmartMotorControllerConfig](smart-motor-controller-config.md)
- [SparkWrapper](spark-wrapper.md)
- [C++ SmartMotorController](../../cpp/motor-controllers/smart-motor-controller.md)
